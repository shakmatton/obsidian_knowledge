# Singleton e Sistema de UI no Unity
### Notas de estudo — Space Shooter (XR / Top-Down)

---

## 1. O que é um Singleton?

Um Singleton é um **padrão de projeto** (design pattern) que garante que uma classe tenha **apenas uma instância** em toda a aplicação, e que essa instância seja **acessível de qualquer lugar** do código.

No Unity, é muito usado para sistemas globais como:
- Gerenciador de UI (UI_Manager)
- Gerenciador de áudio
- Gerenciador de pontuação
- Gerenciador de cenas

**Sem Singleton:** você precisa arrastar referências no Inspector para cada script que precisar falar com o UI_Manager.

**Com Singleton:** qualquer script chama `UI_Manager.Instance.ShowGameOver()` diretamente, sem nenhuma referência no Inspector.

---

## 2. Propriedade com `get` e `private set`

```csharp
public static UI_Manager Instance { get; private set; }
```

Isso é uma **propriedade** do C#. Diferente de um campo simples, ela permite controlar **quem pode ler** e **quem pode escrever** o valor.

| Parte | Significado |
|---|---|
| `public` | qualquer código pode **ler** |
| `static` | pertence à **classe**, não a um objeto específico |
| `get` | permite leitura de fora da classe |
| `private set` | só código **dentro da própria classe** pode atribuir o valor |

### Por que isso importa?

```csharp
// PERMITIDO em qualquer script — só está lendo
UI_Manager.Instance.ShowGameOver();

// PROIBIDO em scripts externos — tentativa de escrita
UI_Manager.Instance = outraCoisa; // erro de compilação
```

Isso **protege o Singleton**: nenhum script externo pode acidentalmente substituir a instância registrada.

---

## 3. Como o Singleton se registra — `Awake()`

```csharp
private void Awake()
{
    if (Instance == null)
    {
        Instance = this;
        DontDestroyOnLoad(gameObject);
    }
    else
    {
        Destroy(gameObject);
    }
}
```

### Linha por linha:

**`Awake()`**
Chamado pelo Unity antes do `Start()`, assim que o objeto é criado. É o lugar certo para inicializar o Singleton — garante que ele existe antes de qualquer outro script tentar usá-lo.

**`if (Instance == null)`**
Verifica se já existe uma instância registrada. Se não existe, este objeto se registra como a instância oficial.

**`Instance = this`**
`this` = o objeto atual que está executando o script. Ele se auto-registra na variável estática da classe.

**`DontDestroyOnLoad(gameObject)`**
Faz o objeto sobreviver quando uma nova cena é carregada. Útil se o UI_Manager precisar persistir entre cenas (menus, gameplay, créditos etc).

**`else { Destroy(gameObject); }`**
Se já existe um UI_Manager registrado (por exemplo, ao carregar uma nova cena que também tem um UI_Manager), o segundo é destruído imediatamente. Garante que só existe **um**.

### ⚠️ Erro comum: `Destroy(this)` vs `Destroy(gameObject)`

```csharp
Destroy(this);        // destrói só o componente (script) — objeto continua na cena
Destroy(gameObject);  // destrói o objeto inteiro — CORRETO para o Singleton
```

---

## 4. Como chamar o Singleton de outro script

```csharp
UI_Manager.Instance.ShowGameOver();
```

Separando em partes:

```
UI_Manager   .   Instance   .   ShowGameOver()
    ^                ^                ^
 a classe        variável         método que
 (onde a         estática que     você quer
 Instance        guarda o         executar
 fica)           objeto real
```

O prefab (asteroide, tiro) **não vira** um UI_Manager. Ele continua sendo o que é. Ele apenas **pergunta para a classe onde está a instância**, e chama o método nessa instância.

É equivalente a ter uma referência direta, mas sem precisar arrastá-la no Inspector:

```csharp
// Com Singleton (sem Inspector)
UI_Manager.Instance.ShowGameOver();

// Sem Singleton (precisaria do [SerializeField])
[SerializeField] private UI_Manager uiManager;
uiManager.ShowGameOver();
```

---

## 5. O código completo explicado

### `UI_Manager.cs`

```csharp
public static UI_Manager Instance { get; private set; }
```
Variável estática que guarda a referência para o único UI_Manager da cena. `private set` impede que scripts externos a sobrescrevam.

---

```csharp
[Header("Telas de UI")]
[SerializeField] private GameObject gameOverScreen;
[SerializeField] private GameObject victoryScreen;
[SerializeField] private GameObject creditsScreen;
[SerializeField] private GameObject statsScreen;
[SerializeField] private GameObject introScreen;
```
`[Header()]` cria um título visual no Inspector para organizar os campos.
Os campos são `private` (boa prática), mas `[SerializeField]` os expõe no Inspector mesmo assim.
Cada campo recebe um Panel/Canvas filho arrastado via Inspector.

---

```csharp
private void Start()
{
    HideAllScreens();
    ShowIntro();
}
```
`Start()` roda depois de todos os `Awake()`, então o Singleton já está configurado. Esconde todas as telas e exibe só a de introdução ao iniciar o jogo.

---

```csharp
public void ShowGameOver()
{
    HideAllScreens();
    gameOverScreen.SetActive(true);
}
```
Sempre chama `HideAllScreens()` primeiro para garantir que nenhuma tela anterior fique visível ao mesmo tempo. Depois ativa só a tela desejada. O mesmo padrão se repete para todas as outras telas.

---

```csharp
private void HideAllScreens()
{
    gameOverScreen.SetActive(false);
    victoryScreen.SetActive(false);
    creditsScreen.SetActive(false);
    statsScreen.SetActive(false);
    introScreen.SetActive(false);
}
```
`private` porque é um detalhe interno — nenhum script externo precisa chamar isso diretamente.
`SetActive(false)` desativa o GameObject: ele fica invisível e inativo na cena.

---

### `PrefabDestroyer.cs`

```csharp
[SerializeField] private bool triggerGameOver;
```
Configurado no Inspector. Permite reusar o mesmo script com comportamentos diferentes:
- **Asteroide:** `triggerGameOver = true`, `autoDestroy = true`
- **Tiro:** `triggerGameOver = false`, `autoDestroy = true`

---

```csharp
private void OnTriggerEnter(Collider other)
{
    if (!tagsToDestroy.Contains(other.tag)) return;
    // ...
}
```
`OnTriggerEnter` é chamado quando dois Colliders se sobrepõem (sem física de rigidbody).
A condição com `!` e `return` é um **early return**: sai cedo se a tag não for relevante, evitando aninhamento desnecessário de `if`.

---

```csharp
if (triggerGameOver)
    UI_Manager.Instance.ShowGameOver();
```
Comunicação com o Singleton. Sem referência no Inspector, sem `[SerializeField]`. O script pergunta para a classe onde está sua instância, e chama o método diretamente.

---

## 6. Setup na cena Unity

1. Crie um **GameObject vazio** chamado `UI_Manager` na cena
2. Anexe o script `UI_Manager.cs` a ele
3. Crie um **Canvas** com Panels filhos para cada tela (GameOver, Victory, Credits...)
4. Arraste cada Panel no campo correspondente do Inspector do UI_Manager
5. No `PrefabDestroyer` do asteroide: `triggerGameOver = true`, `autoDestroy = true`
6. No `PrefabDestroyer` do tiro: `triggerGameOver = false`, `autoDestroy = true`

---

## 7. Resumo dos conceitos

| Conceito | O que faz |
|---|---|
| `static` | pertence à classe, não a uma instância |
| `{ get; private set; }` | leitura pública, escrita só interna |
| `Awake()` | executa antes do `Start()`, ideal para inicialização |
| `DontDestroyOnLoad` | objeto sobrevive à troca de cenas |
| `Destroy(gameObject)` | destrói o objeto inteiro (não só o componente) |
| `SetActive(bool)` | ativa ou desativa um GameObject na cena |
| Early return (`if (!x) return`) | sai cedo de um método para evitar aninhamento |

---

## 8. Extensões futuras possíveis

- Passar **dados para a tela** (ex: pontuação final no Game Over):
  ```csharp
  public void ShowGameOver(int score)
  {
      HideAllScreens();
      gameOverScreen.SetActive(true);
      scoreText.text = "Pontuação: " + score;
  }
  ```

- Chamar o Singleton a partir do **sistema de pontuação** para mostrar tela de vitória quando todos os asteroides forem destruídos:
  ```csharp
  UI_Manager.Instance.ShowVictory();
  ```

- Usar o mesmo UI_Manager para **pausar o jogo**:
  ```csharp
  public void ShowPause()
  {
      HideAllScreens();
      pauseScreen.SetActive(true);
      Time.timeScale = 0f; // pausa a física e animações
  }
  ```
