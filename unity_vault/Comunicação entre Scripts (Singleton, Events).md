# Comunicação entre Scripts — Sistema de Score (Unity)

## Visão geral do fluxo

O objetivo era: **tiro destrói asteroide → score sobe → UI atualiza na tela.**

Isso envolve 4 scripts, cada um com uma responsabilidade bem definida:

```
PrefabDestroyer.cs  →  detecta a colisão e dispara um evento (OnCollide)
SpaceBullet.cs       →  escuta esse evento e avisa o ScoreManager
ScoreManager.cs      →  guarda o score (Singleton) e avisa quem quiser saber que ele mudou
UI_ScoreDisplay.cs   →  escuta o ScoreManager e atualiza o texto na tela
```

Fluxo completo, passo a passo:

```
1. Bala colide com asteroide
2. PrefabDestroyer.OnTriggerEnter() detecta a colisão
3. PrefabDestroyer dispara o evento OnCollide
4. SpaceBullet estava inscrito em OnCollide → executa OnBulletCollide()
5. OnBulletCollide() chama ScoreManager.Instance.HandleBulletScore()
6. HandleBulletScore() incrementa o score e dispara o evento OnScoreChanged
7. UI_ScoreDisplay estava inscrito em OnScoreChanged → executa UpdateScore()
8. UpdateScore() atualiza o texto TMP na tela
```

Repare que existem **dois mecanismos diferentes** sendo usados de propósito:

- **Evento + inscrição** (`PrefabDestroyer → SpaceBullet` e `ScoreManager → UI_ScoreDisplay`): usado quando quem dispara o aviso **não precisa conhecer** quem vai reagir a ele. `PrefabDestroyer` não sabe (nem precisa saber) que existe um `SpaceBullet`; `ScoreManager` não sabe que existe uma UI. Isso é bom para desacoplar sistemas.
- **Chamada direta via Singleton** (`SpaceBullet → ScoreManager`): usado quando um script *precisa* acessar outro de forma direta e há só uma instância dele na cena inteira (só existe 1 `ScoreManager`). Em vez de arrastar uma referência no Inspector, qualquer script chama `ScoreManager.Instance.MetodoQualquer()`.

---

## Problema 1 — `ScoreManager` não avisava a UI

**O que estava errado:**
No código original, `HandleBulletScore()` incrementava a variável `_acumulador`, mas não existia nenhum evento sendo disparado. Havia um método `ChangeScore(int score)` completamente vazio, que era claramente uma tentativa de ser "o evento que a UI escuta" — mas um método vazio não avisa ninguém, ele só existe se for chamado, e ninguém o chamava.

**Por que isso quebrava o fluxo:**
Mesmo que todo o resto funcionasse perfeitamente, a variável `_acumulador` subia "escondida" dentro do `ScoreManager` e nenhum outro script tinha como saber que ela tinha mudado. Faltava literalmente o elo entre "o score mudou" e "avise quem estiver interessado".

**Solução aplicada:**
```csharp
public event Action<int> OnScoreChanged;

public void HandleBulletScore()
{
    _acumulador++;
    OnScoreChanged?.Invoke(_acumulador);   // avisa a UI que o score mudou
    ...
}
```
Isso segue exatamente o mesmo padrão que já existia entre `PrefabDestroyer` e `SpaceBullet`: um evento que carrega o valor novo do score (`Action<int>`), disparado com `?.Invoke(...)` — o `?.` evita erro caso ninguém esteja inscrito ainda (se não tiver ninguém ouvindo, simplesmente não faz nada, em vez de dar `NullReferenceException`).

---

## Problema 2 — Misturando dois padrões de comunicação ao mesmo tempo

**O que estava errado:**
No código antigo (que você mantém comentado como referência — boa prática, aliás), existiam ao mesmo tempo:
- Uma referência arrastada `[SerializeField] private SpaceBullet spaceBullet;`
- Uma linha comentada `spaceBullet.OnBulletCollide += HandleBulletScore;`
- E, simultaneamente, o `SpaceBullet` já chamando `ScoreManager.Instance.HandleBulletScore()` diretamente.

**Por que isso é perigoso (mesmo comentado):**
Se as duas formas de comunicação estivessem ativas ao mesmo tempo, `HandleBulletScore()` seria chamado **duas vezes** por colisão: uma vez pela chamada direta do Singleton, e outra pela inscrição no evento. Resultado: o score subiria de 2 em 2 em vez de 1 em 1 — um bug sutil, difícil de perceber só olhando a tela (o jogo "funciona", só que errado).

**Solução aplicada:**
Removi o campo `spaceBullet` e a inscrição comentada do `ScoreManager` atual, deixando só **uma via de comunicação**: a chamada direta via Singleton (`SpaceBullet → ScoreManager.Instance`). É importante, ao revisar sistemas de eventos, sempre perguntar: "por quantos caminhos diferentes esse método pode ser chamado?" — se a resposta for mais de um, geralmente é bug.

---

## Problema 3 — `UI_ScoreDisplay.cs` não existia

Ele era só mencionado nos comentários dos outros scripts, mas nunca tinha sido criado. Criei o script do zero, seguindo o padrão de inscrição em evento:

```csharp
public class UI_ScoreDisplay : MonoBehaviour
{
    [SerializeField] private TextMeshProUGUI scoreText;

    private void Start()
    {
        if (ScoreManager.Instance != null)
        {
            ScoreManager.Instance.OnScoreChanged += UpdateScore;
            UpdateScore(0);
        }
    }

    private void UpdateScore(int newScore)
    {
        scoreText.text = newScore.ToString();
    }

    private void OnDestroy()
    {
        if (ScoreManager.Instance != null)
            ScoreManager.Instance.OnScoreChanged -= UpdateScore;
    }
}
```

**Por que a inscrição acontece em `Start()` e não em `Awake()` ou `OnEnable()`:**
O Unity garante que **todos** os `Awake()` da cena rodam antes de **qualquer** `Start()`. Como o Singleton `ScoreManager.Instance` é atribuído dentro do `Awake()` dele, usar `Start()` no `UI_ScoreDisplay` garante que, na hora de tentar se inscrever, o `ScoreManager.Instance` já existe — não importa a ordem em que os objetos aparecem na Hierarchy. Se a inscrição fosse feita em `OnEnable()`, haveria risco de rodar antes do `Awake()` do `ScoreManager`, gerando `NullReferenceException` (o mesmo tipo de erro que você teve depois, só que por outro motivo).

**Por que existe o `OnDestroy()` desinscrevendo:**
Sempre que um script se inscreve (`+=`) em um evento, ele deveria se desinscrever (`-=`) quando for destruído. Se isso não for feito, o evento fica com uma referência "fantasma" para um objeto que não existe mais, o que pode causar `MissingReferenceException` na próxima vez que o evento for disparado (o C# ainda tenta chamar o método de um objeto já destruído). Isso segue o mesmo cuidado que já existia em `SpaceBullet.OnDestroy()`.

---

## Problema 4 — `NullReferenceException` em `SpaceBullet.cs` (linha 19)

**O que estava acontecendo:**
O erro `Object reference not set to an instance of an object` na linha `prefabDestroyer.OnCollide += OnBulletCollide;` significa uma coisa só: a variável `prefabDestroyer` estava `null` naquele momento.

**Por que ela estava `null`:**
`[SerializeField]` faz o campo **aparecer** no Inspector, mas não o **preenche** sozinho. É preciso arrastar manualmente o objeto/componente correto para aquele campo, dentro do prefab. Como isso não tinha sido feito (ou tinha sido perdido em algum momento, ex: reimportação do prefab, remoção acidental da referência etc.), o campo ficou vazio, e o código tentou usar algo que não existia.

**Como você resolveu:**
Você encontrou o campo vazio no Inspector do prefab e arrastou a referência correta (o componente `PrefabDestroyer`, que fica no mesmo GameObject) — foi isso que resolveu.

**Reforço que adicionei no código, para o futuro:**
```csharp
private void Awake()
{
    if (prefabDestroyer == null)
    {
        prefabDestroyer = GetComponent<PrefabDestroyer>();
    }

    if (prefabDestroyer == null)
    {
        Debug.LogError($"SpaceBullet em '{gameObject.name}': PrefabDestroyer não encontrado...");
    }
}
```
Isso **não substitui** o ato de arrastar no Inspector — é um "plano B" automático. Como os dois scripts (`PrefabDestroyer` e `SpaceBullet`) sempre vivem no mesmo GameObject dentro do prefab, `GetComponent<PrefabDestroyer>()` consegue achar o componente sozinho, sem precisar de configuração manual. E, caso nem isso funcione (por exemplo, se algum dia o `PrefabDestroyer` for removido do prefab por engano), o `Debug.LogError` avisa exatamente qual objeto está com problema, em vez de deixar o Unity estourar uma exception genérica.

---

## Resumo — por que funciona agora

| Ligação | Mecanismo | Onde é "plugado" |
|---|---|---|
| `PrefabDestroyer` → `SpaceBullet` | Evento (`Action`) | `SpaceBullet.Start()`, via `prefabDestroyer.OnCollide += OnBulletCollide` |
| `SpaceBullet` → `ScoreManager` | Chamada direta (Singleton) | `SpaceBullet.OnBulletCollide()`, via `ScoreManager.Instance.HandleBulletScore()` |
| `ScoreManager` → `UI_ScoreDisplay` | Evento (`Action<int>`) | `UI_ScoreDisplay.Start()`, via `ScoreManager.Instance.OnScoreChanged += UpdateScore` |

A regra geral que fica de lição:
- **Eventos** = bons para "avisar sem saber quem está ouvindo" (baixo acoplamento).
- **Singleton com chamada direta** = bom para "sei que só existe 1 instância disso no jogo inteiro e preciso chamar algo nela diretamente".
- **Toda inscrição em evento precisa ser desfeita** quando o objeto morre (`OnDestroy`), senão sobra referência fantasma.
- **`[SerializeField]` não preenche sozinho** — é sempre preciso arrastar no Inspector, ou ter um fallback via `GetComponent` para os casos em que os dois scripts estão garantidamente no mesmo objeto.
