# Guia Completo: Configuração de Jump e Sprint em Unity XR

## Índice
1. [Visão Geral do Sistema](#visão-geral-do-sistema)
2. [Configuração do Jump Provider](#configuração-do-jump-provider)
3. [Configuração do Gravity Provider](#configuração-do-gravity-provider)
4. [Gerenciamento de Input Actions](#gerenciamento-de-input-actions)
5. [Implementação de Sprint](#implementação-de-sprint)
6. [Rigidbody e Física em VR](#rigidbody-e-física-em-vr)
7. [Layers e Detecção de Colisão](#layers-e-detecção-de-colisão)
8. [Arquitetura e Boas Práticas](#arquitetura-e-boas-práticas)

---

## Visão Geral do Sistema

### Componentes Principais do XR Origin

```
VR Scene
└── XR Origin (VR)
    ├── Camera Offset
    │   └── Main Camera (onde está a cabeça do player)
    ├── Locomotion Mediator
    └── Locomotion
        ├── Move
        │   └── Continuous Move Provider
        ├── Turn
        │   └── Snap Turn Provider / Continuous Turn Provider
        ├── Gravity
        │   └── Gravity Provider
        └── Jump
            └── Jump Provider
```

### Pacotes Necessários
- **XR Interaction Toolkit** (versão 3.0+)
- **Input System** (novo sistema de input do Unity)
- **Starter Assets** (samples do XR Interaction Toolkit)

---

## Configuração do Jump Provider

### Requisitos Obrigatórios

O **Jump Provider** depende do **Gravity Provider** para funcionar. Sem o Gravity Provider, o Jump Provider será automaticamente desabilitado.

```csharp
// Trecho do código do Jump Provider
protected override void Awake()
{
    base.Awake();
    m_HasGravityProvider = ComponentLocatorUtility<GravityProvider>.TryFindComponent(out m_GravityProvider);
    if (!m_HasGravityProvider)
    {
        Debug.LogError("Could not find Gravity Provider component which is required by the Jump Provider component. Disabling component.", this);
        enabled = false;
        return;
    }
}
```

### Adicionando os Componentes

1. **Selecione o XR Origin** no Hierarchy
2. **Add Component** → **Gravity Provider** (obrigatório primeiro)
3. **Add Component** → **Jump Provider**

### Configurações Recomendadas do Jump Provider

```yaml
Jump Provider:
  Jump Height: 1.25                    # Altura em metros
  Variable Height Jump: ✓              # Permite controlar altura segurando o botão
  Min Jump Hold Time: 0.1              # Tempo mínimo de pulo
  Max Jump Hold Time: 0.5              # Tempo máximo de pulo
  Early Out Deceleration Speed: 0.1    # Velocidade de desaceleração ao soltar
  In Air Jump Count: 0                 # Pulos no ar (comece com 0)
  Unlimited In Air Jumps: ☐            # Pulos ilimitados no ar (desmarcar)
  Jump Forgiveness Window: 0.25        # Coyote time em segundos
  Disable Gravity During Jump: ☐       # Desmarcar para física realista
```

### Importando Starter Assets

1. **Window** → **Package Manager**
2. Localize **XR Interaction Toolkit**
3. Na seção **Samples**, importe **Starter Assets**
4. Isso cria o asset **XRI Default Input Actions** com ações pré-configuradas

### Configurando o Jump Input

No Inspector do Jump Provider:

```yaml
Jump Input:
  Input Source Mode: Input Action Reference
  Input Action Reference: XRI RightHand Locomotion/Jump
```

**Observação:** O binding padrão para testes no editor pode ser a tecla "1" em vez de teclas mais intuitivas como Space ou Shift.

---

## Configuração do Gravity Provider

### Papel do Gravity Provider

O Gravity Provider é responsável por:
- Aplicar gravidade ao player
- Detectar se o player está no chão (grounded)
- Gerenciar a velocidade de queda (terminal velocity)
- Controlar quando a gravidade é pausada (durante pulos, teleportes, etc.)

### Detecção de Chão (Ground Detection)

O Gravity Provider usa **SphereCast** para detectar o chão:

```csharp
// Lança uma esfera da cabeça do player para baixo
m_IsGrounded = m_LocalPhysicsScene.SphereCast(
    GetBodyHeadPosition(),           // Posição da cabeça
    m_SphereCastRadius,              // Raio da esfera (0.09m)
    -GetCurrentUp(),                 // Direção (para baixo)
    m_GroundedAllocHits,             // Array de resultados
    GetLocalHeadHeight(),            // Distância máxima
    m_SphereCastLayerMask,           // Layers a detectar
    m_SphereCastTriggerInteraction   // Ignorar triggers
) > 0;
```

### Configurações Recomendadas

```yaml
Gravity Provider:
  Use Gravity: ✓
  Use Local Space Gravity: ☐           # Desmarcar para começar (gravidade sempre para baixo)
  Terminal Velocity: 90                 # Velocidade máxima de queda
  Gravity Acceleration Modifier: 1      # Multiplicador de aceleração
  Update Character Controller Center Each Frame: ✓
  
  # Ground Detection
  Sphere Cast Radius: 0.09              # Raio da detecção (9cm)
  Sphere Cast Distance Buffer: -0.05    # Margem de erro (negativo = mais tolerante)
  Sphere Cast Layer Mask: Everything exceto Ignore Raycast
  Sphere Cast Trigger Interaction: Ignore
```

### Visualização de Debug

Quando o XR Origin está selecionado no modo Play, você verá gizmos:
- **Esfera verde** = player está no chão (grounded)
- **Esfera vermelha** = player no ar

---

## Gerenciamento de Input Actions

### Estrutura do XRI Default Input Actions

```
XRI Default Input Actions
├── XRI Head
├── XRI Left (controlador esquerdo)
├── XRI Left Interaction
├── XRI Left Locomotion
│   ├── Turn
│   ├── Move
│   └── Jump
├── XRI Right (controlador direito)
├── XRI Right Interaction
└── XRI Right Locomotion
    ├── Turn
    ├── Move
    └── Jump
```

### Diferença entre XRI e Player Input Actions

**XRI RightHand Locomotion/Jump (RECOMENDADO para VR):**
- Otimizado para controladores VR
- Mapeia automaticamente botões dos controles (Quest, Index, Vive)
- Funciona out-of-the-box com headsets VR

**Player/Jump (NÃO recomendado para VR):**
- Input genérico para jogos não-VR
- Funciona no teclado mas não está configurado para VR
- Use apenas para projetos desktop

### Como Modificar Input Actions

#### Abrindo o Editor

1. Localize **XRI Default Input Actions** no Project
2. Clique duplo para abrir o **Input Actions Editor**

#### Estrutura da Janela

```
┌─────────────────┬─────────────────┬──────────────────────┐
│  Action Maps    │    Actions      │  Binding Properties  │
│  (esquerda)     │    (centro)     │     (direita)        │
├─────────────────┼─────────────────┼──────────────────────┤
│ XRI Right       │ Jump            │ Binding              │
│ Locomotion      │ ├─ Primary      │ Path: ...            │
│                 │ │   Button      │                      │
│                 │ └─ <Keyboard>/1 │ Interactions: ...    │
└─────────────────┴─────────────────┴──────────────────────┘
```

#### Adicionando/Modificando Bindings de Teclado

**Método 1: Modificar binding existente**

1. Expanda a ação **Jump** (clique na setinha)
2. Selecione o binding `<Keyboard>/1`
3. No painel direito, clique em **Path**
4. Procure por **Keyboard → Left Shift** ou **Space**
5. Selecione a tecla
6. **Save Asset**

**Método 2: Adicionar novo binding**

1. Clique com botão direito em **Jump**
2. Selecione **Add Binding**
3. Com o novo binding selecionado:
   - Clique em **Path**
   - Navegue: **Keyboard → Left Shift**
4. **Save Asset**

**Método 3: Listen Mode (mais rápido)**

1. Selecione o binding
2. Clique em **Listen** (ícone de microfone/ouvido)
3. Pressione a tecla desejada no teclado
4. O binding será configurado automaticamente
5. **Save Asset**

### Estrutura Final Ideal para Jump

```
Jump
├── PrimaryButton [RightHand XR Controller]  # Botão A/X do controle VR
└── <Keyboard>/leftShift                     # Para testes no editor
```

**Ambos coexistem perfeitamente:**
- No editor sem VR: usa o teclado
- Com headset VR: usa o botão do controle

### Habilitando Input Actions no Runtime

Adicione o **Input Action Manager** ao XR Origin:

1. Selecione **XR Origin**
2. **Add Component** → **Input Action Manager**
3. No campo **Action Assets**, adicione **XRI Default Input Actions**

Isso garante que as ações estejam habilitadas quando o jogo iniciar.

---

## Implementação de Sprint

### Conceitos de Sprint em VR

**Opção 1: Modificador de velocidade (mais comum)**
- Segurar um botão aumenta a velocidade
- Soltar o botão volta à velocidade normal
- Mais intuitivo em VR

**Opção 2: Toggle**
- Pressionar uma vez para correr
- Pressionar novamente para andar

**Opção 3: Gatilho progressivo**
- Velocidade proporcional à pressão no analógico
- Mais imersivo mas complexo

### Criando a Input Action para Sprint

1. Abra **XRI Default Input Actions**
2. Navegue até **XRI Right Locomotion**
3. Adicione nova Action:
   - Nome: **Sprint**
   - Action Type: **Button**
   - Control Type: **Button**

4. Adicione bindings:
   ```
   Sprint
   ├── <XRController>{RightHand}/gripPressed    # Grip do controle VR
   └── <Keyboard>/leftCtrl                      # Ctrl para testes
   ```

5. **Save Asset**

### Script SprintModifier.cs

```csharp
using UnityEngine;
using UnityEngine.XR.Interaction.Toolkit.Locomotion.Movement;
using UnityEngine.XR.Interaction.Toolkit.Inputs.Readers;

namespace UnityEngine.XR.Interaction.Toolkit.Locomotion
{
    /// <summary>
    /// Modifica a velocidade do Continuous Move Provider baseado em input de sprint.
    /// </summary>
    public class SprintModifier : MonoBehaviour
    {
        [Header("Referências")]
        [SerializeField]
        [Tooltip("O Continuous Move Provider que será modificado.")]
        ContinuousMoveProviderBase m_MoveProvider;

        [Header("Configurações de Velocidade")]
        [SerializeField]
        [Tooltip("Velocidade normal de caminhada.")]
        float m_WalkSpeed = 2.5f;

        [SerializeField]
        [Tooltip("Velocidade ao correr (sprint).")]
        float m_SprintSpeed = 5f;

        [Header("Input")]
        [SerializeField]
        [Tooltip("Input que ativa o sprint quando pressionado.")]
        XRInputButtonReader m_SprintInput = new XRInputButtonReader("Sprint");

        /// <summary>
        /// Velocidade normal de caminhada.
        /// </summary>
        public float walkSpeed
        {
            get => m_WalkSpeed;
            set => m_WalkSpeed = value;
        }

        /// <summary>
        /// Velocidade ao correr.
        /// </summary>
        public float sprintSpeed
        {
            get => m_SprintSpeed;
            set => m_SprintSpeed = value;
        }

        /// <summary>
        /// Input do sprint.
        /// </summary>
        public XRInputButtonReader sprintInput
        {
            get => m_SprintInput;
            set => m_SprintInput = value;
        }

        void OnEnable()
        {
            // Habilita o input action
            m_SprintInput.EnableDirectActionIfModeUsed();

            // Se não foi atribuído manualmente, tenta encontrar o Move Provider
            if (m_MoveProvider == null)
                m_MoveProvider = GetComponent<ContinuousMoveProviderBase>();

            if (m_MoveProvider == null)
            {
                Debug.LogError("Continuous Move Provider não encontrado! Atribua manualmente no Inspector.", this);
                enabled = false;
            }
        }

        void OnDisable()
        {
            // Desabilita o input action
            m_SprintInput.DisableDirectActionIfModeUsed();
        }

        void Update()
        {
            if (m_MoveProvider == null)
                return;

            // Verifica se o botão de sprint está pressionado
            bool isSprinting = m_SprintInput.ReadIsPerformed();

            // Aplica a velocidade correspondente
            m_MoveProvider.moveSpeed = isSprinting ? m_SprintSpeed : m_WalkSpeed;
        }

        void OnValidate()
        {
            // Garante que walk speed não seja maior que sprint speed
            if (m_WalkSpeed > m_SprintSpeed)
                m_WalkSpeed = m_SprintSpeed;
        }
    }
}
```

### Explicação dos Métodos Lifecycle

#### OnEnable()
Chamado quando o componente é habilitado (início do jogo ou quando ativado).

**Responsabilidades:**
- Habilitar input actions
- Buscar referências a outros componentes
- Inscrever-se em eventos

```csharp
void OnEnable()
{
    // Habilita o input action serializado
    m_SprintInput.EnableDirectActionIfModeUsed();
    
    // Busca automática de componente no mesmo GameObject
    if (m_MoveProvider == null)
        m_MoveProvider = GetComponent<ContinuousMoveProviderBase>();
}
```

#### OnDisable()
Chamado quando o componente é desabilitado (fim do jogo ou quando desativado).

**Responsabilidades:**
- Desabilitar input actions
- Desinscrever-se de eventos
- Limpar recursos

```csharp
void OnDisable()
{
    // Desabilita o input action
    m_SprintInput.DisableDirectActionIfModeUsed();
}
```

#### Update()
Chamado todo frame (cerca de 60-90 vezes por segundo).

**Responsabilidades:**
- Lógica principal do componente
- Leitura de inputs
- Atualização de estados

```csharp
void Update()
{
    // Lê o estado do input
    bool isSprinting = m_SprintInput.ReadIsPerformed();
    
    // Atualiza a velocidade baseada no input
    m_MoveProvider.moveSpeed = isSprinting ? m_SprintSpeed : m_WalkSpeed;
}
```

#### OnValidate()
Chamado quando valores são modificados no Inspector (apenas no Editor).

**Responsabilidades:**
- Validar valores de configuração
- Aplicar regras de negócio
- Prevenir configurações inválidas

```csharp
void OnValidate()
{
    // Previne que velocidade de caminhada seja maior que corrida
    if (m_WalkSpeed > m_SprintSpeed)
        m_WalkSpeed = m_SprintSpeed;
}
```

### Instalação e Configuração

#### 1. Criar o Script

```
Project
└── Assets
    └── Scripts
        └── SprintModifier.cs
```

#### 2. Adicionar ao GameObject Move

```
Locomotion
└── Move
    ├── Continuous Move Provider
    └── Sprint Modifier (adicionar aqui)
```

**Por que no mesmo GameObject?**
- O SprintModifier **modifica** o Continuous Move Provider
- `GetComponent()` busca componentes no mesmo GameObject
- Organização lógica: tudo relacionado a movimento junto

#### 3. Configurar no Inspector

```yaml
Sprint Modifier:
  Move Provider: (autodetetado - Continuous Move Provider)
  Walk Speed: 2.5
  Sprint Speed: 5.0
  Sprint Input:
    Input Source Mode: Input Action Reference
    Input Action Reference: XRI RightHand Locomotion/Sprint
```

### Valores Recomendados de Velocidade

```yaml
Walk Speed: 2.0 - 2.5    # Caminhada realista
Sprint Speed: 4.0 - 5.0  # Corrida (cuidado com motion sickness!)
```

**Importante em VR:** Velocidades muito altas podem causar enjoo (motion sickness). Comece com valores conservadores e teste.

---

## Rigidbody e Física em VR

### Rigidbody no XR Origin: NÃO ADICIONAR

**Por que NÃO usar Rigidbody no XR Origin?**

1. **Conflito com Locomotion System**
   - Locomotion Providers movem o player via **transformações diretas**
   - Rigidbody usa **física do Unity**
   - Ambos tentariam controlar a posição → movimento irregular

2. **Tracking VR**
   - A posição do player é determinada pelo **tracking do headset**
   - Rigidbody poderia interferir com isso

3. **Desnecessário**
   - O player VR não precisa de física simulada
   - Colisões são gerenciadas pelo Character Controller

### Quando Usar Rigidbody

**Use Rigidbody APENAS em:**
- Objetos grabbables (que o player pode pegar)
- Props que caem/rolam/colidem
- Objetos que precisam de física simulada
- Projéteis

**Exemplo:**
```
Cubo Interativo
├── Box Collider
├── Rigidbody            # Para física
└── XR Grab Interactable # Para ser pegável
```

### Character Controller

O XR Origin **pode (e geralmente deve)** ter um **Character Controller**:

```yaml
Character Controller:
  Center: (0, 1, 0)      # Altura do torso
  Radius: 0.3            # Largura do player
  Height: 2.0            # Altura total
  Skin Width: 0.08       # Margem de colisão
```

**Vantagens:**
- Colisões robustas com paredes/objetos
- Player não atravessa geometria
- Melhor controle sobre movimento
- Não usa física complexa (mais performático que Rigidbody)

**Como adicionar:**
1. Selecione **XR Origin** (ou o GameObject filho "Origin")
2. **Add Component** → **Character Controller**

### Objetos Estáticos (Chão, Paredes, Mesas)

**Configuração correta:**
```
Chão
├── Mesh Collider (ou Box Collider)  # Apenas collider
└── (NÃO adicionar Rigidbody)        # Objetos estáticos não precisam
```

**Objetos estáticos só precisam de Colliders:**
- Mesh Collider para geometria complexa
- Box/Sphere/Capsule Collider para geometria simples
- **Sem Rigidbody** → mais performático

---

## Layers e Detecção de Colisão

### Por Que XR Origin Usa "Ignore Raycast"

**Problema sem "Ignore Raycast":**
```
Gravity Provider faz SphereCast para baixo
    ↓
Acerta o próprio XR Origin
    ↓
Falso positivo: "sempre grounded"
    ↓
Player não consegue pular ou cair
```

**Solução com "Ignore Raycast":**
```
Gravity Provider faz SphereCast para baixo
    ↓
Ignora o próprio XR Origin
    ↓
Detecta apenas chão/objetos externos
    ↓
Física funciona corretamente
```

### Configuração de Layers

#### XR Origin (VR)
```yaml
Layer: Ignore Raycast
Componentes:
  - Transform
  - Character Controller (colisões físicas com paredes)
  - Locomotion Mediator
  - Gravity Provider
  - Jump Provider
```

#### Chão e Objetos Sólidos
```yaml
Layer: Default (ou custom como "Environment")
Componentes:
  - Mesh Collider / Box Collider
  - (sem Rigidbody se for estático)
```

#### XR Interaction Simulator
```yaml
Layer: Ignore Raycast
```
**Motivo:** Objetos virtuais de simulação (mãos/cabeça simuladas) não devem ser detectados por raycasts de interação.

### Sphere Cast Layer Mask (Gravity Provider)

No **Gravity Provider**, o campo **Sphere Cast Layer Mask** controla quais layers são consideradas "chão":

```yaml
Sphere Cast Layer Mask:
  ✓ Default
  ✓ Environment (se você criou essa layer)
  ✓ Props
  ☐ Ignore Raycast      # NÃO marcar
  ☐ UI                  # NÃO marcar
```

**Por padrão:** `Everything` exceto `Ignore Raycast`

### Sphere Cast Trigger Interaction

```yaml
Sphere Cast Trigger Interaction: Ignore
```

**Opções:**
- **Ignore:** Triggers não contam como chão (recomendado)
- **Collide:** Triggers contam como chão
- **UseGlobal:** Usa configuração global do Physics

**Exemplo prático:**
- Zona de teleporte (trigger) → não deve contar como chão
- Plataforma sólida (collider) → deve contar como chão

---

## Arquitetura e Boas Práticas

### Padrão Composite (Composição)

Unity usa intensivamente o padrão **Composite**:
- **GameObject** = Container de componentes
- **Component** = Comportamento/funcionalidade individual

```
GameObject "Move"
├── Transform (sempre presente)
├── Continuous Move Provider (comportamento base)
└── Sprint Modifier (comportamento adicional)
```

**Vantagens:**
- Modularidade: cada componente tem uma responsabilidade
- Reusabilidade: componentes podem ser combinados de formas diferentes
- Facilidade de manutenção: mudanças em um componente não afetam outros

### Padrão dos Locomotion Providers

Todos os Locomotion Providers seguem a mesma estrutura:

```csharp
public class AlgumProvider : LocomotionProvider
{
    // Campos serializados (aparecem no Inspector)
    [SerializeField] float m_AlgumaPropriedade;
    
    // Propriedades públicas (acessíveis via código)
    public float algunmaPropriedade
    {
        get => m_AlgumaPropriedade;
        set => m_AlgumaPropriedade = value;
    }
    
    // Lifecycle methods
    protected virtual void Awake() { /* Buscar referências */ }
    protected virtual void OnEnable() { /* Inscrever eventos, habilitar inputs */ }
    protected virtual void OnDisable() { /* Desinscrever eventos, desabilitar inputs */ }
    protected virtual void Update() { /* Lógica principal */ }
}
```

**Consistência:**
- Fácil de entender novos providers
- Previsível onde encontrar funcionalidades
- Facilita debugging

### Organização de Hierarquia

```
XR Origin (VR)
└── Locomotion
    ├── Move        # Movimento horizontal
    ├── Turn        # Rotação
    ├── Gravity     # Gravidade vertical
    └── Jump        # Pulo
```

**Princípio:** Cada GameObject tem **um único propósito de locomoção**

**Sprint não é uma nova forma de locomoção**, é uma **modificação** do movimento existente. Por isso vai como componente adicional em "Move", não como novo GameObject.

### GetComponent e Referências

#### GetComponent() - Busca no mesmo GameObject

```csharp
void OnEnable()
{
    // Busca no MESMO GameObject
    m_MoveProvider = GetComponent<ContinuousMoveProviderBase>();
}
```

**Vantagens:**
- Rápido (não percorre hierarquia)
- Fortalece acoplamento lógico (componentes relacionados juntos)
- Autodeteção de referências

#### GetComponentInChildren() - Busca em filhos

```csharp
void Awake()
{
    // Busca no GameObject e em todos os filhos
    m_Camera = GetComponentInChildren<Camera>();
}
```

#### GetComponentInParent() - Busca em pais

```csharp
void Start()
{
    // Busca no GameObject e em todos os pais
    m_XROrigin = GetComponentInParent<XROrigin>();
}
```

### Serialização e Inspector

#### Campos Serializados

```csharp
[SerializeField]
[Tooltip("Descrição que aparece no Inspector")]
float m_WalkSpeed = 2.5f;
```

**Características:**
- `[SerializeField]` torna campo privado visível no Inspector
- Valor padrão é aplicado quando componente é adicionado
- Mudanças no Inspector sobrescrevem valor no código

#### Propriedades Públicas

```csharp
public float walkSpeed
{
    get => m_WalkSpeed;
    set => m_WalkSpeed = value;
}
```

**Quando usar:**
- Acesso via código de outros scripts
- Validação de valores
- Triggers de eventos ao mudar valor

### Input Reader Pattern

O XR Interaction Toolkit usa o padrão **Input Reader**:

```csharp
[SerializeField]
XRInputButtonReader m_SprintInput = new XRInputButtonReader("Sprint");

void OnEnable()
{
    m_SprintInput.EnableDirectActionIfModeUsed();
}

void Update()
{
    bool isPressed = m_SprintInput.ReadIsPerformed();
}
```

**Vantagens:**
- Abstração: não precisa saber se é Input Action ou Input Value
- Configurável no Inspector
- Fácil de testar com diferentes inputs

### Lifecycle de MonoBehaviour

Ordem de execução dos métodos:

```
Instantiate GameObject
    ↓
Awake()          # Primeira inicialização, buscar componentes
    ↓
OnEnable()       # Habilitar inputs, inscrever eventos
    ↓
Start()          # Inicialização que depende de outros Awake()
    ↓
Update()         # Loop principal, todo frame
Update()
Update()
...
    ↓
OnDisable()      # Desabilitar inputs, desinscrever eventos
    ↓
OnDestroy()      # Limpeza final
```

**Boas práticas:**
- **Awake:** Buscar referências internas (GetComponent)
- **OnEnable:** Habilitar features, inscrever eventos
- **Start:** Inicialização que depende de outros objetos
- **Update:** Lógica que roda todo frame
- **OnDisable:** Reverter tudo feito em OnEnable
- **OnDestroy:** Limpeza de recursos pesados

---

## Troubleshooting Comum

### Jump não funciona

**Sintomas:** Apertar botão de jump não faz nada

**Checklist:**
1. ✓ Gravity Provider está presente e habilitado?
2. ✓ Jump Provider está presente e habilitado?
3. ✓ Jump Input está configurado corretamente?
4. ✓ Input Action Manager tem o XRI Default Input Actions?
5. ✓ Character Controller ou Collider está presente no XR Origin?
6. ✓ Chão tem Collider e está na layer mask correta?

**Console errors comuns:**
- "Could not find Gravity Provider" → Adicionar Gravity Provider
- "Camera is not set in XR Origin" → Verificar referência da câmera

### Player atravessa objetos

**Sintomas:** Player passa através de paredes/chão

**Soluções:**
1. Adicionar **Character Controller** ao XR Origin
2. Verificar que objetos têm **Colliders**
3. Conferir **Sphere Cast Layer Mask** no Gravity Provider
4. Garantir que XR Origin não tem Rigidbody

### Sprint não funciona

**Sintomas:** Apertar botão de sprint não muda velocidade

**Checklist:**
1. ✓ Sprint Input Action foi criada?
2. ✓ Sprint bindings foram configurados?
3. ✓ SprintModifier está no GameObject "Move"?
4. ✓ Move Provider está referenciado no SprintModifier?
5. ✓ Sprint Input no SprintModifier aponta para a action correta?

**Debug:**
```csharp
void Update()
{
    bool isSprinting = m_SprintInput.ReadIsPerformed();
    Debug.Log($"Is Sprinting: {isSprinting}");  // Verificar se input está sendo lido
    
    m_MoveProvider.moveSpeed = isSprinting ? m_SprintSpeed : m_WalkSpeed;
    Debug.Log($"Current Speed: {m_MoveProvider.moveSpeed}");  // Verificar se velocidade muda
}
```

### Gravity não funciona

**Sintomas:** Player não cai quando no ar

**Checklist:**
1. ✓ Use Gravity está marcado?
2. ✓ Sphere Cast Layer Mask inclui layer do chão?
3. ✓ Chão tem Collider?
4. ✓ XR Origin está em "Ignore Raycast"?
5. ✓ Camera está configurada no XR Origin?

**Visualizar detecção de chão:**
- Selecione XR Origin no modo Play
- Observe gizmos: verde = grounded, vermelho = no ar

### Motion Sickness em VR

**Sintomas:** Usuário sente enjoo/náusea ao usar VR

**Causas comuns:**
- Velocidades de movimento muito altas
- Aceleração/desaceleração abrupta
- Framerate baixo (< 72 FPS)
- Movimento não alinhado com olhar

**Soluções:**
1. Reduzir velocidades (Walk: 2.0, Sprint: 4.0)
2. Adicionar vignette ao mover (escurecer bordas da tela)
3. Usar teleporte em vez de movimento contínuo
4. Garantir framerate estável (90+ FPS)
5. Adicionar ponto de referência fixo (ex: nariz virtual)

---

## Glossary / Glossário

### Termos de XR/VR

**XR Origin:** GameObject raiz que representa o espaço do player em VR. Contém a câmera (headset) e sistema de tracking.

**Headset:** Dispositivo de realidade virtual usado na cabeça (Meta Quest, Valve Index, etc.)

**Controller:** Controle manual usado para interagir em VR (normalmente dois, um para cada mão)

**Tracking:** Sistema que detecta posição e rotação do headset e controllers no espaço 3D.

**Locomotion:** Sistema de movimento do player (andar, pular, teleportar, etc.)

**Grounded:** Estado em que o player está tocando o chão.

**Coyote Time (Jump Forgiveness Window):** Período após sair do chão onde ainda é possível pular, tornando controles mais responsivos.

### Termos de Unity

**GameObject:** Entidade básica na cena do Unity. Container de componentes.

**Component:** Módulo de funcionalidade que pode ser anexado a um GameObject.

**Transform:** Componente obrigatório que define posição, rotação e escala de um GameObject.

**Collider:** Componente que define a forma física de um objeto para colisões.

**Rigidbody:** Componente que adiciona física simulada (gravidade, forças, colisões) a um objeto.

**Character Controller:** Componente especializado para movimento de personagens, sem física complexa.

**Layer:** Sistema de categorização de GameObjects para controlar interações (colisões, raycasts, etc.)

**Layer Mask:** Filtro que especifica quais layers devem ser consideradas em uma operação.

**Raycast:** Lançar um raio invisível para detectar objetos.

**SphereCast:** Similar a raycast, mas usa uma esfera em vez de um raio fino.

**Serialization:** Processo de salvar estado de objetos (valores no Inspector são serializados).

**Inspector:** Painel no Unity que mostra e permite editar componentes de um GameObject selecionado.

**Hierarchy:** Painel que mostra todos os GameObjects na cena em estrutura de árvore.

**Project:** Painel que mostra todos os assets (arquivos) do projeto.

### Termos de Input System

**Input Action:** Ação abstrata (jump, move, shoot) que pode ser mapeada para diferentes inputs.

**Input Action Asset:** Arquivo que contém coleção de Input Actions organizadas.

**Action Map:** Grupo de Input Actions relacionadas (ex: "Locomotion", "Combat").

**Binding:** Conexão entre uma Input Action e um controle físico (botão, tecla, analógico).

**Input Action Reference:** Referência serializada a uma Input Action específica.

**Input Source Mode:** Como o input é fornecido (Input Action, Input Value manualmente configurado, etc.)

**Path:** Caminho que identifica um controle específico (ex: `<Keyboard>/space`).

**Performed:** Estado de uma action quando foi executada (botão pressionado, trigger puxado).

**Started:** Início da interação com um controle.

**Canceled:** Fim da interação (botão solto).

### Termos de Programação

**Namespace:** Agrupamento lógico de código para evitar conflitos de nomes.

**Class:** Modelo/template que define estrutura e comportamento de objetos.

**Method:** Função que pertence a uma classe.

**Property:** Encapsulamento de um campo com getter/setter.

**Field:** Variável que armazena dados em uma classe.

**Public:** Acessível de qualquer lugar.

**Private:** Acessível apenas dentro da própria classe.

**Protected:** Acessível na classe e em classes derivadas.

**Virtual:** Método que pode ser sobrescrito em classes derivadas.

**Override:** Sobrescrever implementação de método virtual em classe base.

**Inheritance:** Classe que herda características de outra classe.

**Interface:** Contrato que define métodos que uma classe deve implementar.

**Event:** Sistema de notificação para comunicação entre objetos.

**Delegate:** Tipo que representa referência a métodos.

**Callback:** Método chamado em resposta a um evento.

**Lifecycle Methods:** Métodos chamados automaticamente pelo Unity (Awake, Start, Update, etc.)

---

## Recursos Adicionais

### Documentação Oficial

- [Unity XR Interaction Toolkit Manual](https://docs.unity3d.com/Packages/com.unity.xr.interaction.toolkit@latest)
- [Input System Documentation](https://docs.unity3d.com/Packages/com.unity.inputsystem@latest)
- [Unity Scripting Reference](https://docs.unity3d.com/ScriptReference/)

### Tutoriais Recomendados

- **Valem Tutorials** (YouTube) - Tutoriais de XR/VR no Unity
- **Unity Learn** - Cursos oficiais gratuitos
- **Brackeys** (YouTube) - Fundamentos de Unity e C#

### Comunidades

- Unity Forums - XR & VR section
- Reddit - r/Unity3D, r/virtualreality
- Discord - Unity Developer Community

### Ferramentas de Debug

**Console Window:** Ver logs, warnings e erros

```csharp
Debug.Log("Mensagem informativa");
Debug.LogWarning("Aviso");
Debug.LogError("Erro");
```

**Scene View Gizmos:** Visualizações no editor

```csharp
void OnDrawGizmos()
{
    Gizmos.color = Color.red;
    Gizmos.DrawWireSphere(transform.position, 1f);
}
```

**Profiler:** Analisar performance

- Window → Analysis → Profiler
- Ver CPU, GPU, memória, física em tempo real

**Frame Debugger:** Analisar renderização frame a frame

- Window → Analysis → Frame Debugger

---

## Notas de Versão

Este guia foi criado baseado em:
- **Unity:** 6.2
- **XR Interaction Toolkit:** 3.0+
- **Input System:** 1.7+
- **Data:** Janeiro 2026

Algumas funcionalidades podem variar em versões diferentes. Sempre consulte a documentação oficial para sua versão específica.

---

## Changelog do Guia

### v1.0 - Janeiro 2026
- Guia inicial completo
- Cobertura de Jump Provider
- Cobertura de Gravity Provider  
- Implementação de Sprint
- Explicação de Rigidbody e Layers
- Arquitetura e boas práticas
- Troubleshooting
- Glossário completo

---

**Autor:** Baseado em conversas sobre desenvolvimento XR/VR no Unity  
**Licença:** Livre para uso pessoal e educacional