# Unity XR - Coroutines, IEnumerator e Eventos

> Guia completo sobre Coroutines no Unity, eventos XR Interaction Toolkit e comparações com outros paradigmas de programação.

---

## 📋 Índice

1. [Código Completo da Estrela](#código-completo-da-estrela)
2. [Inicialização de Arrays com Chaves](#inicialização-de-arrays-com-chaves)
3. [Coroutines - Conceito Fundamental](#coroutines---conceito-fundamental)
4. [Eventos XR: selectEntered vs activated](#eventos-xr-selectentered-vs-activated)
5. [IEnumerator - A Interface das Coroutines](#ienumerator---a-interface-das-coroutines)
6. [yield return - Pausando a Execução](#yield-return---pausando-a-execução)
7. [Coroutines vs Threads](#coroutines-vs-threads)
8. [Comparações com Outras Linguagens](#comparações-com-outras-linguagens)
9. [Resumo e Boas Práticas](#resumo-e-boas-práticas)

---

## 🌟 Código Completo da Estrela

```csharp
using System.Collections;
using UnityEngine;
using UnityEngine.XR.Interaction.Toolkit.Interactables;

public class Star : MonoBehaviour
{
    private XRGrabInteractable grabInteractable;
    private Renderer starRenderer;
    private Material starMaterial;
    
    [SerializeField] private bool superStar = false;
    [SerializeField] private float colorChangeSpeed = 0.1f;
    
    // Array de cores para piscar (estilo Super Mario)
    private Color[] rainbowColors = new Color[]
    {
        Color.red,
        new Color(1f, 0.5f, 0f),    // laranja
        Color.yellow,
        Color.green,
        Color.cyan,
        Color.blue,
        new Color(0.5f, 0f, 1f),    // roxo
        Color.magenta
    };
    
    private Color originalColor;
    private Coroutine blinkCoroutine;
    
    void Start()
    {
        grabInteractable = GetComponent<XRGrabInteractable>();
        starRenderer = GetComponent<Renderer>();
        starMaterial = starRenderer.material;
        originalColor = starMaterial.color;
        
        grabInteractable.selectEntered.AddListener(OnGrabbed);
        grabInteractable.selectExited.AddListener(OnReleased);
    }

    void OnGrabbed(SelectEnterEventArgs args)
    {
        if (!superStar)
        {
            superStar = true;
            
            if (blinkCoroutine != null)
                StopCoroutine(blinkCoroutine);
                
            blinkCoroutine = StartCoroutine(BlinkColors());
        }
    }
    
    void OnReleased(SelectExitEventArgs args)
    {
        // Opcional: parar de piscar quando soltar
        // StopBlinking();
    }
    
    IEnumerator BlinkColors()
    {
        int colorIndex = 0;
        
        while (superStar)
        {
            starMaterial.color = rainbowColors[colorIndex];
            
            if (starMaterial.HasProperty("_EmissionColor"))
            {
                starMaterial.SetColor("_EmissionColor", rainbowColors[colorIndex] * 2f);
            }
            
            colorIndex = (colorIndex + 1) % rainbowColors.Length;
            yield return new WaitForSeconds(colorChangeSpeed);
        }
    }
    
    public void StopBlinking()
    {
        superStar = false;
        
        if (blinkCoroutine != null)
        {
            StopCoroutine(blinkCoroutine);
            blinkCoroutine = null;
        }
        
        starMaterial.color = originalColor;
        
        if (starMaterial.HasProperty("_EmissionColor"))
        {
            starMaterial.SetColor("_EmissionColor", Color.black);
        }
    }
    
    void OnDestroy()
    {
        if (starMaterial != null)
            Destroy(starMaterial);
    }
}
```

---

## 🎨 Inicialização de Arrays com Chaves

### Sintaxe Completa

```csharp
private Color[] rainbowColors = new Color[] 
{ 
    Color.red,
    new Color(1f, 0.5f, 0f),  // laranja customizado
    Color.yellow
};
```

### Anatomia da Declaração

| Parte | Descrição |
|-------|-----------|
| `Color[]` | Tipo do array (array de cores) |
| `new Color[]` | Cria nova instância do array |
| `{ ... }` | Inicializador - define valores diretamente |

### Cores Nativas vs Customizadas

**✅ Cores Nativas do Unity:**
```csharp
Color.red      // (1, 0, 0)
Color.green    // (0, 1, 0)
Color.blue     // (0, 0, 1)
Color.yellow   // (1, 1, 0)
Color.cyan     // (0, 1, 1)
Color.magenta  // (1, 0, 1)
Color.white    // (1, 1, 1)
Color.black    // (0, 0, 0)
Color.gray     // (0.5, 0.5, 0.5)
```

**🎨 Cores Customizadas (criadas manualmente):**
```csharp
new Color(1f, 0.5f, 0f)     // Laranja (R=1, G=0.5, B=0)
new Color(0.5f, 0f, 1f)     // Roxo (R=0.5, G=0, B=1)
new Color(1f, 0.75f, 0.8f)  // Rosa claro
```

> **Nota:** Valores RGB no Unity vão de `0.0` a `1.0` (não 0-255 como em outras plataformas)

---

## ⚙️ Coroutines - Conceito Fundamental

### O Problema que Coroutines Resolvem

#### ❌ Abordagem Errada (Bloqueia o Jogo)

```csharp
void MudarCores()
{
    while(true)
    {
        material.color = Color.red;
        Thread.Sleep(1000);  // ❌ TRAVA TODO O UNITY!
        material.color = Color.blue;
    }
}
```

**Problema:** `Thread.Sleep()` bloqueia a thread principal, congelando o jogo inteiro.

#### ✅ Abordagem Correta (Com Coroutine)

```csharp
IEnumerator MudarCores()
{
    while(true)
    {
        material.color = Color.red;
        yield return new WaitForSeconds(1f);  // ✅ Pausa, mas jogo continua!
        material.color = Color.blue;
        yield return new WaitForSeconds(1f);
    }
}
```

**Solução:** A coroutine pausa naquele ponto, permite o Unity processar frames, e depois retoma.

### Como Funciona Internamente

```
Frame 1:
  ├─ Executa até o primeiro yield
  ├─ material.color = Color.red
  └─ "Unity, me acorde em 1 segundo"

Frame 2-60:
  └─ Coroutine dormindo... (jogo rodando normal)

Frame 60 (1 segundo depois):
  ├─ Unity: "Já passou 1 segundo!"
  ├─ Retoma após o yield
  ├─ material.color = Color.blue
  └─ "Unity, me acorde em 1 segundo"

Frame 61-120:
  └─ Coroutine dormindo...

Frame 120:
  └─ Retoma novamente...
```

### Anatomia de uma Coroutine

```csharp
// 1. Declarar com tipo IEnumerator
IEnumerator MinhaCorrotina()
{
    Debug.Log("Início");
    
    // 2. Usar yield para pausar
    yield return new WaitForSeconds(2f);
    
    Debug.Log("2 segundos depois");
    
    // 3. Pode pausar múltiplas vezes
    yield return null;  // Espera 1 frame
    
    Debug.Log("1 frame depois");
}

// 4. Iniciar com StartCoroutine
void Start()
{
    StartCoroutine(MinhaCorrotina());
}
```

### Guardando Referência da Coroutine

```csharp
private Coroutine blinkCoroutine;  // Guarda referência

void OnGrabbed()
{
    // Para a anterior se estiver rodando
    if (blinkCoroutine != null)
        StopCoroutine(blinkCoroutine);
    
    // Inicia nova e guarda referência
    blinkCoroutine = StartCoroutine(BlinkColors());
}
```

**Por que verificar `!= null`?**
- Evita múltiplas coroutines rodando simultaneamente
- Previne comportamentos duplicados (cores piscando em dobro)

---

## 🎮 Eventos XR: selectEntered vs activated

### Hierarquia de Eventos XR Grab

```
PEGAR OBJETO (Grab)
    ↓
selectEntered → Dispara quando agarra o objeto
    ↓
APERTAR GATILHO (Trigger)
    ↓
activated → Dispara quando aperta trigger (objeto já na mão)
    ↓
SOLTAR OBJETO (Release)
    ↓
selectExited → Dispara quando solta o objeto
```

### Tabela Comparativa

| Evento | Quando Dispara | Uso Comum | Exemplo |
|--------|----------------|-----------|---------|
| `selectEntered` | Ao **agarrar** objeto | Feedback imediato ao pegar | Estrela piscar ao pegar |
| `activated` | Ao **apertar trigger** (já segurando) | Ação secundária | Ligar/desligar lanterna |
| `selectExited` | Ao **soltar** objeto | Cleanup, reset | Parar efeitos sonoros |

### Exemplos Práticos

#### Exemplo 1: Estrela (selectEntered)

```csharp
// Queremos que pisque IMEDIATAMENTE ao pegar
grabInteractable.selectEntered.AddListener(OnGrabbed);

void OnGrabbed(SelectEnterEventArgs args)
{
    StartCoroutine(BlinkColors());  // Começa a piscar
}
```

**Por que `selectEntered`?** 
- Não precisa apertar nada extra
- Pegar = efeito automático (como no Super Mario)

#### Exemplo 2: Luminária (activated)

```csharp
// Queremos ligar/desligar apertando gatilho
grabInteractable.activated.AddListener(OnActivated);

void OnActivated(ActivateEventArgs args)
{
    LightStatus = !LightStatus;  // Toggle luz
}
```

**Por que `activated`?**
- Precisa de controle: liga/desliga quando quiser
- Pegar ≠ ligar automaticamente

#### Exemplo 3: Arma (ambos)

```csharp
// Feedback ao pegar + atirar com trigger
grabInteractable.selectEntered.AddListener(OnGrabbed);
grabInteractable.activated.AddListener(OnShoot);

void OnGrabbed(SelectEnterEventArgs args)
{
    PlayPickupSound();  // Som ao pegar
}

void OnActivated(ActivateEventArgs args)
{
    Fire();  // Atira quando aperta trigger
}
```

### SelectEnterEventArgs vs ActivateEventArgs

```csharp
void OnGrabbed(SelectEnterEventArgs args)
{
    // Informações disponíveis:
    args.interactorObject;    // Qual mão pegou (esquerda/direita)
    args.interactableObject;  // Qual objeto foi pego
}

void OnActivated(ActivateEventArgs args)
{
    // Informações disponíveis:
    args.interactorObject;    // Qual mão ativou
    args.interactableObject;  // Qual objeto foi ativado
}
```

**Quando usar essas informações?**
- Diferenciar mão esquerda/direita
- Aplicar força/feedback específico por objeto
- Logs e debug

---

## 🔄 IEnumerator - A Interface das Coroutines

### O que é IEnumerator?

`IEnumerator` é uma **interface do C#** que:
1. Permite iterar sobre coleções (`foreach`)
2. **No Unity:** permite pausar e retomar execução

### Não é sobre "enumerar cores"!

```csharp
// ❌ Conceito ERRADO
IEnumerator BlinkColors()  // NÃO é "enumerador de cores"
{
    // É um método que pode PAUSAR
}

// ✅ Conceito CORRETO
IEnumerator BlinkColors()  // Método pausável
{
    yield return ...  // PAUSA aqui
}
```

### Sintaxe Obrigatória

```csharp
// ❌ ERRO - Método normal não pode usar yield
void MinhaFuncao()
{
    yield return new WaitForSeconds(1f);  // ❌ Erro de compilação!
}

// ✅ CORRETO - IEnumerator permite yield
IEnumerator MinhaCorrotina()
{
    yield return new WaitForSeconds(1f);  // ✅ Funciona!
}
```

> **Regra:** Todo método que usa `yield` DEVE retornar `IEnumerator`

### IEnumerator em Outros Contextos (C# Puro)

```csharp
// Uso tradicional: iterar sobre coleção customizada
IEnumerator<int> ContarAte5()
{
    yield return 1;
    yield return 2;
    yield return 3;
    yield return 4;
    yield return 5;
}

// Uso:
foreach (int num in ContarAte5())
{
    Console.WriteLine(num);  // 1, 2, 3, 4, 5
}
```

**Unity adaptou esse conceito** para criar o sistema de coroutines!

---

## ⏸️ yield return - Pausando a Execução

### O que é yield?

`yield` significa **"ceder"** ou **"entregar"** em inglês. No código:
- **Cede controle** de volta ao Unity
- **Entrega** um valor (tempo de espera, condição, etc.)

### Sintaxe

```csharp
yield return <o_que_esperar>;
```

### Tipos de Espera

#### 1. Esperar Tempo (Segundos)

```csharp
yield return new WaitForSeconds(2.5f);  // Espera 2.5 segundos
```

**Usa:** Tempo de jogo (afetado por `Time.timeScale`)

#### 2. Esperar Tempo Real

```csharp
yield return new WaitForSecondsRealtime(2.5f);  // Tempo real
```

**Usa:** Tempo real (NÃO afetado por pausas/slow-motion)

#### 3. Esperar 1 Frame

```csharp
yield return null;  // Próximo frame
```

**Comum em:** Loops de movimento suave

#### 4. Esperar até o Final do Frame

```csharp
yield return new WaitForEndOfFrame();  // Após câmeras renderizarem
```

**Usa:** Screenshots, efeitos pós-processamento

#### 5. Esperar até Condição Verdadeira

```csharp
yield return new WaitUntil(() => player.health > 0);
```

**Continua quando:** `player.health` for maior que 0

#### 6. Esperar enquanto Condição Verdadeira

```csharp
yield return new WaitWhile(() => isLoading);
```

**Continua quando:** `isLoading` se tornar `false`

#### 7. Esperar outra Coroutine

```csharp
yield return StartCoroutine(OutraCorrotina());
```

**Espera:** Outra corrotina terminar completamente

### Exemplo Completo com Múltiplos Yields

```csharp
IEnumerator SequenciaCompleta()
{
    Debug.Log("Início");
    
    // Espera 1 segundo
    yield return new WaitForSeconds(1f);
    Debug.Log("1 segundo passou");
    
    // Espera player aparecer
    yield return new WaitUntil(() => player != null);
    Debug.Log("Player apareceu!");
    
    // Espera 5 frames
    for (int i = 0; i < 5; i++)
    {
        yield return null;
    }
    Debug.Log("5 frames passaram");
    
    // Espera outra corrotina
    yield return StartCoroutine(CarregarRecursos());
    Debug.Log("Recursos carregados!");
}
```

---

## 🧵 Coroutines vs Threads

### Diferenças Fundamentais

| Característica | **Threads** | **Coroutines** |
|----------------|-------------|----------------|
| **Threads simultâneas** | ✅ Múltiplas (paralelismo real) | ❌ Uma só (Main Thread) |
| **Execução** | Paralela | Intercalada (cooperativa) |
| **Acessa Unity API** | ❌ **NÃO** (crash!) | ✅ Sim (seguro) |
| **Complexidade** | 🔴 Alta | 🟢 Baixa |
| **Race Conditions** | ✅ Possível | ❌ Impossível |
| **Sincronização** | Precisa `lock` | Não precisa |
| **Uso no Unity** | 🔴 Evitar | ✅ Recomendado |

### Por que Threads são Perigosos no Unity?

```csharp
// ❌ PERIGO! VAI CRASHAR!
void Start()
{
    new Thread(() => 
    {
        transform.position = Vector3.zero;  // 💥 CRASH!
        // Unity API não é thread-safe!
    }).Start();
}
```

**Problema:** APIs do Unity (`Transform`, `GameObject`, `Material`, etc.) só funcionam na **Main Thread**.

### Quando Usar Threads no Unity?

**✅ Apenas para processamento pesado SEM Unity API:**

```csharp
void Start()
{
    new Thread(() => 
    {
        // ✅ OK - cálculos puros, sem Unity
        for (int i = 0; i < 1000000; i++)
        {
            float resultado = Mathf.Sqrt(i);  // Cálculo matemático
        }
        
    }).Start();
}
```

**⚠️ Mas ainda assim, prefira:**
- `Task` (async/await do C#)
- Job System (Unity Jobs)
- Burst Compiler

### Coroutines: Solução Segura

```csharp
// ✅ SEGURO! Tudo na Main Thread
IEnumerator MoverObjeto()
{
    while (true)
    {
        transform.position += Vector3.forward * Time.deltaTime;
        yield return null;  // Próximo frame
    }
}
```

### Visualização: Thread vs Coroutine

```
THREADS (Paralelismo Real):
━━━━━━━━━━━━━━━━━━━━━━━━  Thread 1 (Main)
    ━━━━━━━━━━━━━━━━━━━  Thread 2 (Worker)
        ━━━━━━━━━━━━━━━  Thread 3 (Worker)

Executam SIMULTANEAMENTE em diferentes cores do CPU


COROUTINES (Intercalado):
━━━━━━━━━━━━━━━━━━━━━━━━  Main Thread ÚNICA
  ▲    ▲    ▲    ▲    ▲
  │    │    │    │    │
  C1   C2   C3   C4   C5   (Coroutines pausam/retomam)

Executam SEQUENCIALMENTE, cedendo controle
```

---

## 🌍 Comparações com Outras Linguagens

### Python - Generators (Mais Parecido!)

```python
# Python - Generators com yield
def meu_gerador():
    print("início")
    yield 1  # Pausa aqui
    print("meio")
    yield 2  # Pausa aqui
    print("fim")

gen = meu_gerador()
next(gen)  # "início", retorna 1
next(gen)  # "meio", retorna 2
next(gen)  # "fim", StopIteration
```

**Similaridade:** `yield` funciona EXATAMENTE igual!

### JavaScript - async/await

```javascript
// JavaScript - Assíncrono
async function exemplo() {
    console.log("início");
    await sleep(1000);  // Pausa 1 segundo
    console.log("1 segundo depois");
}

// Em Unity seria:
IEnumerator exemplo() {
    Debug.Log("início");
    yield return new WaitForSeconds(1f);
    Debug.Log("1 segundo depois");
}
```

**Similaridade:** Conceito de execução não-bloqueante.

### Java - CompletableFuture

```java
// Java - Programação assíncrona
CompletableFuture.supplyAsync(() -> {
    Thread.sleep(1000);
    return "resultado";
}).thenAccept(resultado -> {
    System.out.println(resultado);
});
```

**Diferença:** Java não tem `yield` nativo (apenas em preview desde Java 19).

### C# Moderno - async/await

```csharp
// C# fora do Unity
async Task MinhaFuncao()
{
    Debug.Log("início");
    await Task.Delay(1000);  // Pausa 1 segundo
    Debug.Log("1 segundo depois");
}

// Unity Coroutine equivalente
IEnumerator MinhaFuncao()
{
    Debug.Log("início");
    yield return new WaitForSeconds(1f);
    Debug.Log("1 segundo depois");
}
```

> **Nota:** Unity está adotando `async/await` gradualmente, mas coroutines ainda são o padrão.

---

## 📚 Resumo e Boas Práticas

### A Tríade Unity

```
IEnumerator  →  Tipo de retorno obrigatório para yield
     ↓
yield return →  Palavra-chave que pausa execução
     ↓
StartCoroutine() →  Método que inicia a corrotina
```

### Checklist de Coroutines

- [ ] Método retorna `IEnumerator`
- [ ] Usa `yield return` para pausar
- [ ] Inicia com `StartCoroutine()`
- [ ] Guarda referência se precisar parar depois
- [ ] Para com `StopCoroutine()` quando necessário
- [ ] Limpa referências no `OnDestroy()`

### Quando Usar Cada Evento XR

```csharp
// Efeito imediato ao pegar
selectEntered  →  Estrela, moeda, power-up

// Ação com botão (trigger)
activated      →  Lanterna, arma, ferramenta

// Cleanup ao soltar
selectExited   →  Parar sons, resetar estado
```

### Armadilhas Comuns

#### ❌ Esquecer de Parar Coroutine

```csharp
void OnDestroy()
{
    // ❌ ERRO: Coroutine continua rodando!
}

// ✅ CORRETO
void OnDestroy()
{
    if (blinkCoroutine != null)
        StopCoroutine(blinkCoroutine);
}
```

#### ❌ Múltiplas Coroutines Simultâneas

```csharp
void OnGrabbed()
{
    StartCoroutine(Blink());  // ❌ Cria nova toda vez!
}

// ✅ CORRETO
void OnGrabbed()
{
    if (blinkCoroutine != null)
        StopCoroutine(blinkCoroutine);
    
    blinkCoroutine = StartCoroutine(Blink());
}
```

#### ❌ Usar Thread para Unity API

```csharp
new Thread(() => {
    transform.position = ...;  // ❌ CRASH!
}).Start();

// ✅ CORRETO
IEnumerator Mover() {
    transform.position = ...;  // ✅ Seguro
    yield return null;
}
```

### Dicas de Performance

```csharp
// ❌ Cria nova instância toda frame
while (true)
{
    yield return new WaitForSeconds(0.1f);  // Aloca memória
}

// ✅ Reutiliza instância
WaitForSeconds wait = new WaitForSeconds(0.1f);
while (true)
{
    yield return wait;  // Sem alocação
}
```

### Template Base

```csharp
using System.Collections;
using UnityEngine;

public class MinhaClasse : MonoBehaviour
{
    private Coroutine minhaCoroutine;
    
    void Start()
    {
        minhaCoroutine = StartCoroutine(MinhaRotina());
    }
    
    IEnumerator MinhaRotina()
    {
        while (true)
        {
            // Faz algo...
            
            yield return new WaitForSeconds(1f);
        }
    }
    
    void OnDestroy()
    {
        if (minhaCoroutine != null)
            StopCoroutine(minhaCoroutine);
    }
}
```

---

## 🔗 Referências Úteis

- [Unity Manual - Coroutines](https://docs.unity3d.com/Manual/Coroutines.html)
- [XR Interaction Toolkit Documentation](https://docs.unity3d.com/Packages/com.unity.xr.interaction.toolkit@latest)
- [C# yield keyword](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/statements/yield)

---

**Última atualização:** Fevereiro 2026  
**Versão Unity:** 2022.3+ / Unity 6
