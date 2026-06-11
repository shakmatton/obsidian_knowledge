# Unity 6 — Spaceship Controller (Top-Down 2D)

Notas de desenvolvimento: controles de nave espacial em cena XR com visão Top View, simulando um jogo 2D.

---

## Contexto do Projeto

- Cena de testes no Unity 6 com XR Interaction Toolkit
- Visão **Top View** para simular jogo 2D
- Nave espacial precisa desviar de asteroides
- Objeto `Spaceship` é filho de um Empty chamado `Spacecraft` (criado para ter eixo Y pra cima e Z pra frente)
- Controles mapeados nas teclas **8, 4, 5, 6** do numpad

---

## Problemas no Código Original

O código inicial tinha quatro erros principais:

| Problema | Antes | Depois |
|---|---|---|
| Referência inválida | `SpaceshipController _controller` (script referenciando a si mesmo) | `Rigidbody _rb` ou `PlayerInput _playerInput` |
| Nome da variável | `speed` (inexistente) | `_speed` |
| Tipo de leitura | `ReadValue<Vector2>()` atribuído a `Vector3` sem conversão explícita | Atribuído a `Vector2` corretamente |
| Update loop | `Update()` para física | `FixedUpdate()` quando usando Rigidbody |

---

## Configuração do Input Actions Asset

A action `Move2D` deve ser configurada como **2D Vector Composite**, não como bindings individuais.

### Por que o erro `Cannot read value of type 'Vector2' from control 'KeyControl'`?

Cada tecla isolada é um `KeyControl` que retorna `float`. O `ReadValue<Vector2>()` espera um `Vector2`. O **2D Vector Composite** resolve isso combinando 4 teclas e calculando automaticamente o vetor resultante.

### Configuração correta no Input Actions

```
Action Maps: SpaceshipActionMap
└── Move2D
    ├── Action Type: Value
    ├── Control Type: Vector2
    └── 2D Vector Composite
        ├── Up    → Numpad 8
        ├── Down  → Numpad 5
        ├── Left  → Numpad 4
        └── Right → Numpad 6
```

### Tabela de valores retornados

| Tecla pressionada | Vector2 retornado |
|---|---|
| Numpad 8 (Up)    | (0, 1)  |
| Numpad 5 (Down)  | (0, -1) |
| Numpad 4 (Left)  | (-1, 0) |
| Numpad 6 (Right) | (1, 0)  |

---

## Por que criar um Input Actions Asset próprio

O Unity tem um asset padrão global (`InputSystem_Actions`). Usar esse asset para o Spaceship é arriscado: ele pode ser **resetado ou substituído** por atualizações do XR Interaction Toolkit ou do próprio Input System.

O aviso amarelo no Inspector já alertava para isso:

> *"Project-wide Actions asset is not recommended to be used with Player Input because it is a singleton reference and all action maps are enabled by default."*

| | Asset global | Asset próprio |
|---|---|---|
| Risco de ser resetado por updates | ✅ Sim | ❌ Não |
| Compartilhado com outros sistemas (XR, UI...) | ✅ Sim | ❌ Não |
| Fica na sua pasta, sob seu controle | ❌ Não | ✅ Sim |
| Recomendado pelo Unity para Player Input | ❌ Não | ✅ Sim |

### Como criar o asset próprio

```
Project > botão direito > Create > Input Actions
```

Nomeie como `SpaceshipInputActions` (ou similar), salve na sua pasta (ex: `Assets/MyScripts`).
No Inspector do objeto Spaceship, arraste o novo asset para o campo **Actions** do Player Input.

---

## Control Schemes

É um agrupamento que diz ao Unity *"este conjunto de bindings pertence a este tipo de dispositivo"*. Sem ele, o campo **Default Scheme** no Player Input fica vermelho com `<Not Found>`.

### Como criar

No editor do Input Actions asset, no topo onde aparece **"No Control Schemes"**:

1. Clique em **"No Control Schemes"** > **"Add Control Scheme..."**
2. Nomeie como `Keyboard&Mouse`
3. Em **Required Devices**, clique em **+** e adicione `Keyboard` (e `Mouse` opcionalmente)
4. Clique em **Save**
5. Clique em **Save Asset**

### Vincular as bindings ao scheme

Selecione cada binding (Up, Down, Left, Right) e em **Binding Properties > Use in control scheme**, marque **Keyboard&Mouse**.

---

## As 4 Opções de Behavior do Player Input

### 1. Send Messages

O Unity usa **reflexão** para procurar automaticamente métodos no script pelo nome, sem nenhuma configuração manual no Inspector.

```csharp
// Assinatura obrigatória: void OnNomeDoAction(InputValue value)
void OnMove(InputValue value)
{
    _moveInput = value.Get<Vector2>();
}
```

- ✅ Simples, zero configuração no Inspector
- ✅ Bom para protótipos rápidos
- ❌ Menos performático (usa reflexão em runtime)
- ❌ Assinatura rígida — tem que ser `InputValue`, sem escolha

### 2. Broadcast Messages

Funciona igual ao Send Messages, mas propaga o método para **todos os filhos** do GameObject na hierarquia.

- ✅ Útil quando vários objetos filhos precisam responder ao mesmo input
- ❌ Também usa reflexão
- ❌ Raramente necessário

### 3. Invoke Unity Events

O Player Input **expõe eventos no Inspector** (igual a um `onClick` de botão), e você conecta manualmente qual método quer chamar.

```csharp
// Método deve ser public para aparecer no Inspector
public void OnMove(InputAction.CallbackContext ctx)
{
    if (ctx.performed)
        _moveInput = ctx.ReadValue<Vector2>();
    else if (ctx.canceled)
        _moveInput = Vector2.zero;
}
```

Configuração no Inspector:
- Expanda **Spaceship > Move2D**
- Em **performed**, clique em **+**, arraste o GameObject e selecione o método
- Repita para **canceled**

- ✅ Conexões visíveis no Inspector
- ✅ Pode chamar métodos em **outros objetos/scripts**
- ✅ `CallbackContext` dá acesso à fase do input (started/performed/canceled)
- ❌ Requer configuração manual no Inspector para cada action

### 4. Invoke C# Events

A opção mais explícita: você assina os eventos **diretamente no código** usando o sistema de eventos do C#.

```csharp
private void OnEnable()
{
    _playerInput.actions["Move2D"].performed += OnMove;
    _playerInput.actions["Move2D"].canceled  += OnMove;
}

private void OnDisable()
{
    // SEMPRE desassinar para evitar memory leak
    _playerInput.actions["Move2D"].performed -= OnMove;
    _playerInput.actions["Move2D"].canceled  -= OnMove;
}
```

- ✅ Máximo controle e performance
- ✅ Sem reflexão, sem Inspector — tudo explícito no código
- ✅ Ideal para sistemas complexos
- ❌ Mais verboso
- ❌ Fácil de esquecer o `OnDisable`, causando memory leaks

### Quadro comparativo

| | Configuração | Flexibilidade | Custo |
|---|---|---|---|
| **Send Messages**      | Zero      | Baixa  | Reflexão          |
| **Broadcast Messages** | Zero      | Baixa  | Reflexão + filhos |
| **Invoke Unity Events**| Inspector | Média  | Baixo             |
| **Invoke C# Events**   | Código    | Alta   | Mínimo            |

---

## O que é Reflexão (Reflection)?

É a capacidade do código de **inspecionar e invocar a si mesmo em tempo de execução**.

### Analogia

- **Com reflexão:** você chega numa empresa sem saber nada, pede a lista de todos os funcionários, percorre nome por nome até achar "João", e entrega a mensagem.
- **Sem reflexão:** você já tem o telefone do João salvo — liga direto.

### O que o Unity faz internamente no Send Messages

```csharp
// Simplificado — representa o comportamento interno do Unity
var methods = gameObject.GetComponents<MonoBehaviour>()
    .SelectMany(mb => mb.GetType().GetMethods())
    .Where(m => m.Name == "OnMove");

foreach (var method in methods)
    method.Invoke(target, new object[] { inputValue });
```

### Custo relativo

| | Chamada direta | Reflexão |
|---|---|---|
| Endereço do método conhecido em compilação? | ✅ Sim | ❌ Não |
| Precisa varrer a estrutura do objeto? | ❌ Não | ✅ Sim, toda vez |
| Custo relativo | Mínimo | ~10–50x mais lento |

> Para jogos simples com poucos objetos, esse custo é desprezível na prática.

---

## InputActionReference

Uma forma alternativa de referenciar uma action específica **diretamente no Inspector**, sem depender do componente `PlayerInput`.

```csharp
[SerializeField] private InputActionReference moveAction;
```

No Inspector, um campo `moveAction` aparece — basta expandir o asset `SpaceshipInputActions` e arrastar a action `SpaceshipActionMap/Move2D` para esse campo.

O código de assinatura fica assim:

```csharp
private void OnEnable()
{
    moveAction.action.performed += OnMove;
    moveAction.action.canceled  += OnStop;
}

private void OnDisable()
{
    moveAction.action.performed -= OnMove;
    moveAction.action.canceled  -= OnStop;
}
```

A diferença em relação ao `_playerInput.actions["Move2D"]` é que aqui a referência é **direta e tipada** (sem usar string), o que elimina o risco de erro de digitação no nome da action.

---

## Fases do Input: performed e canceled

São **fases** do ciclo de vida de uma `InputAction` — representam o que está acontecendo com o input naquele momento:

| Fase | Quando dispara |
|---|---|
| `started` | No primeiro frame que o botão é pressionado |
| `performed` | Quando o input atinge seu valor (botão mantido/lido) |
| `canceled` | Quando o botão é solto |

Para teclas simples, `started` e `performed` disparam quase juntos. O que se usa na prática é `performed` (pressionado) e `canceled` (solto).

### O que `+=` e `-=` fazem

São operadores de **assinar** e **desassinar** eventos em C#:

```csharp
// "Quando performed acontecer, avise o método OnMove"
moveAction.action.performed += OnMove;

// "Quando performed acontecer, pare de avisar o método OnMove"
moveAction.action.performed -= OnMove;
```

O evento (`performed`, `canceled`) é a **campainha**. O método (`OnMove`, `OnStop`) é **quem atende**. O `+=` instala o atendente, o `-=` remove.

### OnEnable e OnDisable: o que realmente acontece

`OnEnable` e `OnDisable` disparam quando a **caixinha do GameObject é marcada ou desmarcada** no Inspector. Mas o `+=` dentro do `OnEnable` **não chama o método** — apenas registra que ele deverá ser chamado futuramente.

O `InputAction.CallbackContext ctx` nem existe ainda nesse momento: ele é criado pelo Unity **somente quando uma tecla é de fato pressionada**.

```
OnEnable() dispara (caixinha marcada)
    → += apenas registra OnMove e OnStop como ouvintes
    → nenhum método é chamado, nenhum ctx existe
    → _moveInput continua Vector2.zero (valor padrão)

[usuário pressiona Numpad 8]
    → Unity cria o ctx → dispara performed
    → OnMove(ctx) é chamado pela primeira vez
    → _moveInput = (0, 1)

[usuário solta Numpad 8]
    → canceled dispara
    → OnStop(ctx) é chamado
    → _moveInput = Vector2.zero

OnDisable() dispara (caixinha desmarcada)
    → -= remove OnMove e OnStop como ouvintes
    → nenhum método é chamado
```

---

## Código Final (versão com InputActionReference)

```csharp
using UnityEngine;
using UnityEngine.InputSystem;

namespace MyScripts
{
    public class NewSpaceshipController : MonoBehaviour
    {
        private Vector2 _moveInput;
        [SerializeField] private float _speed = 6f;

        [SerializeField] private InputActionReference moveAction;
        // No Inspector: arraste SpaceshipInputActions > SpaceshipActionMap/Move2D para este campo

        private void OnEnable()
        {
            moveAction.action.performed += OnMove;
            moveAction.action.canceled  += OnStop;
        }

        private void OnDisable()
        {
            moveAction.action.performed -= OnMove;
            moveAction.action.canceled  -= OnStop;
        }

        private void OnStop(InputAction.CallbackContext ctx)
        {
            _moveInput = Vector2.zero;
        }

        private void OnMove(InputAction.CallbackContext ctx)
        {
            _moveInput = ctx.ReadValue<Vector2>();
        }

        private void Update()
        {
            Vector3 movement = new Vector3(_moveInput.x, 0f, _moveInput.y);
            transform.localPosition += movement * _speed * Time.deltaTime;
        }
    }
}
```

---

## Observações e Próximos Passos

### `Time.deltaTime` vs `Time.fixedDeltaTime`

| | Usar com |
|---|---|
| `Time.deltaTime` | `Update()` |
| `Time.fixedDeltaTime` | `FixedUpdate()` |

Misturar os dois causa movimento irregular em framerates variáveis.

### Colisões com asteroides (próxima etapa)

Sem Rigidbody, `transform.localPosition` ignora a física — a nave atravessará os asteroides. Duas opções:

| Abordagem | Como detectar colisão |
|---|---|
| Manter sem Rigidbody | `OnTriggerEnter` com `Is Trigger` no Collider |
| Adicionar Rigidbody | `OnCollisionEnter` com física real |

Para um jogo top-down simples, **trigger é suficiente**: adicione um Collider na nave e nos asteroides e marque **Is Trigger**.
