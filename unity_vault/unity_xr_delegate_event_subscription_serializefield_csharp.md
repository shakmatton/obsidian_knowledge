# Eventos e Subscriptions em Unity 6 e C#

## 1. De onde vem o `+=`?

O `+=` é a sintaxe de **subscription** (inscrição) em eventos do C#. Para entendê-la, é preciso partir do conceito de **delegate**.

### Delegate

Um `delegate` é um tipo que representa uma referência a um método — uma "variável que guarda uma função".

```csharp
public delegate void MinhaAcao(int valor);
MinhaAcao minhaVariavel = AlgumMetodo;
```

### Event

Um `event` é um delegate com proteção: só quem o **declarou** pode dispará-lo (`Invoke`). De fora, só é permitido `+=` e `-=`.

```csharp
// Sem event: qualquer classe pode invocar ou sobrescrever
public Action<int> OnVidaAlterada;

// Com event: protegido — só o dono pode invocar
public event Action<int> OnVidaAlterada;
```

---

## 2. Os tipos de delegate prontos do C#

Não é sempre `Action`. Os três casos principais são:

```csharp
// Sem parâmetros, sem retorno
public event Action OnJogadorMorreu;

// Com parâmetros, sem retorno
public event Action<int> OnVidaAlterada;
public event Action<int, string> OnMensagem;

// Com retorno
public event Func<int> OnPedirVidaAtual;
public event Func<int, bool> OnVerificar;

// Delegate customizado (raramente necessário)
public delegate void MinhaAcao(int vida, bool estaVivo);
public event MinhaAcao OnVidaAlterada;
```

> Na prática, `Action` e `Action<T>` resolvem a grande maioria dos casos em Unity.

### `Action<int>` vs. delegate customizado: qual a diferença?

As duas linhas abaixo são **funcionalmente idênticas**:

```csharp
// Opção A: delegate customizado escrito à mão
public delegate void MyEvent(int i);
public event MyEvent OnCountChanged;

// Opção B: Action<int> — delegate pronto do C#
public event Action<int> OnCountChanged;
```

`Action<int>` já é definido pelo C# como um delegate que recebe um `int` e retorna `void` — exatamente o que `MyEvent` faz. O delegate customizado faz sentido quando você quer dar um **nome semântico ao tipo** que outras partes do código vão referenciar explicitamente. Para a maioria dos casos, `Action<T>` é preferido por ser mais conciso.

---

## 3. Exemplo completo: Jogador e HUD

```csharp
// Emissor: não sabe quem está ouvindo
public class Jogador
{
    public event Action<int> OnVidaAlterada;
    private int _vida = 100;

    public void TomarDano(int dano)
    {
        _vida -= dano;
        OnVidaAlterada?.Invoke(_vida); // dispara para quem estiver inscrito
    }
}

// Ouvinte: se inscreve e reage
public class HUDVida : MonoBehaviour
{
    [SerializeField] private Jogador _jogador;

    private void OnEnable()
    {
        _jogador.OnVidaAlterada += AtualizarHUD;
    }

    private void OnDisable()
    {
        _jogador.OnVidaAlterada -= AtualizarHUD;
    }

    private void AtualizarHUD(int novaVida)
    {
        Debug.Log($"Vida atualizada: {novaVida}");
    }
}
```

---

## 4. O operador `?.` (null-conditional)

Um `event` é `null` quando **ninguém está inscrito**. Chamar `.Invoke()` em `null` gera `NullReferenceException`.

```csharp
// ❌ Crasha se ninguém estiver inscrito
OnVidaAlterada.Invoke(_vida);

// ✅ Se for null, simplesmente não faz nada
OnVidaAlterada?.Invoke(_vida);
```

O `?.` **não é uma decisão de design** ("só disparo se alguém ouvir") — é apenas proteção técnica contra null. O emissor continua não se importando com quem ouve.

```csharp
// Funciona em qualquer referência que pode ser null
string nome = null;
int? tamanho = nome?.Length;   // não crasha, retorna null
jogador?.TomarDano(10);        // só chama se jogador != null
```

---

## 5. Onde inscrever e desinscrever

### `Start` vs. `OnEnable`: qual a diferença?

| | `Start` | `OnEnable` |
|---|---|---|
| Quantas vezes roda? | **Uma vez** | Toda vez que o objeto é ativado |
| Objeto desativado e reativado? | Não re-inscreve | Inscreve/desinscreve corretamente |

Para objetos que nunca são desativados e reativados durante o jogo, `Start` funciona bem. O padrão `OnEnable/OnDisable` é mais robusto e cobre casos como pooling de objetos.

### Regra de ouro: sempre desinscreva

| Onde inscrever | Onde desinscrever |
|---|---|
| `OnEnable` | `OnDisable` |
| `Start` | `OnDestroy` |
| `Awake` | `OnDestroy` |

Se o objeto ouvinte for destruído antes do emissor sem desinscrever, o evento tentará chamar um método em um objeto destruído. Quando ambos vivem na mesma cena e são destruídos juntos, o risco é baixo — mas desinscrever é sempre boa prática.

```csharp
// Padrão Start/OnDestroy
void Start()
{
    counter.OnCountChanged += HandleScore;
}

private void OnDestroy()
{
    counter.OnCountChanged -= HandleScore;
}

// Padrão OnEnable/OnDisable (preferido para objetos que podem ser ativados/desativados)
private void OnEnable()
{
    counter.OnCountChanged += HandleScore;
}

private void OnDisable()
{
    counter.OnCountChanged -= HandleScore;
}
```

---

## 6. Fluxo do padrão de eventos

```
Jogador.TomarDano(30)
    └─> OnVidaAlterada?.Invoke(70)
            ├─> HUDVida.AtualizarHUD(70)      ← inscrito
            ├─> SistemaDeAudio.TocarSom(70)   ← inscrito
            └─> GameOver.Verificar(70)         ← inscrito
```

O emissor **não sabe** quem está ouvindo. Qualquer objeto pode se inscrever sem que o emissor precise ser modificado.

---

## 7. Exemplo prático: BigButton, Counter e CounterDisplay

### Diagrama de responsabilidades

```
[Usuário aperta o botão]
        │
        ▼
BigButton.OnSelectEntered()
        │  chama
        ▼
Counter.Increment()
        │  dispara
        ▼
OnCountChanged?.Invoke(count)
        │
        ▼
CounterDisplay.AtualizarDisplay(count)   ← inscrito, recebe o valor
```

> **Counter não sabe que BigButton ou CounterDisplay existem.**

### Counter.cs — dono do evento

```csharp
using System;
using UnityEngine;

public class Counter : MonoBehaviour
{
    private int count = 0;

    // Opção A: delegate customizado
    public delegate void MyEvent(int i);
    public event MyEvent OnCountChanged;

    // Opção B: equivalente, mais conciso (preferido)
    // public event Action<int> OnCountChanged;

    public void Increment()
    {
        count++;
        OnCountChanged?.Invoke(count);
    }
}
```

### BigButton.cs — chama Increment(), não sabe do evento

```csharp
using UnityEngine;
using UnityEngine.XR.Interaction.Toolkit;
using UnityEngine.XR.Interaction.Toolkit.Interactables;
using UnityEngine.XR.Interaction.Toolkit.Interactors;

public class BigButton : XRSimpleInteractable
{
    [SerializeField] private Counter counter;
    private IXRSelectInteractor interactor;

    protected override void OnSelectEntered(SelectEnterEventArgs args)
    {
        base.OnSelectEntered(args);
        interactor = args.interactorObject;
        counter.Increment();
    }

    protected override void OnSelectExited(SelectExitEventArgs args)
    {
        base.OnSelectExited(args);
        interactor = null;
    }
}
```

### CounterDisplay.cs — ouvinte, inscreve-se no evento

```csharp
using TMPro;
using UnityEngine;

public class CounterDisplay : MonoBehaviour
{
    [SerializeField] private Counter counter;

    private TextMeshProUGUI textMeshProUGUI;

    void Start()
    {
        counter.OnCountChanged += HandleScore;
        textMeshProUGUI = GetComponent<TextMeshProUGUI>();
        // GetComponent só funciona para componentes no mesmo GameObject.
        // Para objetos externos, use [SerializeField].
    }

    private void OnDestroy()
    {
        counter.OnCountChanged -= HandleScore;
    }

    private void HandleScore(int count)
    {
        textMeshProUGUI.text = count.ToString();
    }
}
```

---

## 8. Erros comuns

### Linha de execução fora de método

```csharp
public class BigButton : XRSimpleInteractable
{
    public event Action<int> OnCountChanged;

    // ❌ ERRO: statement fora de qualquer método
    OnCountChanged?.Invoke(count);
}
```

O disparo deve estar **dentro** de um método.

### Tentar invocar ou ler retorno de um evento de fora

```csharp
// ❌ Não pode invocar evento de fora da classe dona
_jogador.OnVidaAlterada.Invoke(0);

// ❌ Evento não retorna valor
count = BigButton.OnCountChanged();
```

---

## 9. Quando usar `[SerializeField]` vs `GetComponent`

| Abordagem | Quando usar |
|---|---|
| `[SerializeField]` + arrastar no Inspector | Referências estáveis entre GameObjects sem parentesco garantido |
| `GetComponent<T>()` | Componente no **mesmo** GameObject |
| `GetComponentInParent/Children<T>()` | Componente em parente/filho direto e previsível |
| `GameObject.Find()` | Evitar — frágil, lento, hardcoded |

> Se você se pegar encadeando `GetComponent` de forma torturada para atravessar a hierarquia, é sinal de que um evento provavelmente resolve o acoplamento que você estava tentando criar na mão.
