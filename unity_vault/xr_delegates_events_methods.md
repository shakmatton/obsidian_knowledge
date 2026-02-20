# Unity XR - Sistema de Luminária com Toggle (Guia Completo)

> Material de estudo sobre implementação de sistema de iluminação interativa em Unity XR
> Data: 03 de Fevereiro de 2026

---

## 📑 Índice

1. [Contexto Inicial](#contexto-inicial)
2. [Análise da Conversa Anterior](#análise-da-conversa-anterior)
3. [Solução Implementada](#solução-implementada)
4. [Entendendo Delegates](#entendendo-delegates)
5. [Controle via Inspector](#controle-via-inspector)
6. [Métodos Especiais do Unity](#métodos-especiais-do-unity)
7. [Código Final Completo](#código-final-completo)

---

## Contexto Inicial

### Objetivo do Projeto
Criar uma luminária interativa em Unity XR com dois comportamentos:
- **Luminária 1**: Luz acende enquanto segura o Trigger, apaga ao soltar
- **Luminária 2**: Luz funciona como interruptor (toggle) - apertar liga, apertar novamente desliga

### Componentes Envolvidos
- **GameObject**: Lamp (objeto pai)
- **Component**: Light (filho do Lamp, tipo Spot Light)
- **Script**: Lamp.cs (controla o comportamento)
- **XR Component**: XR Grab Interactable (permite pegar e interagir com o objeto)

---

## Análise da Conversa Anterior

### Problemas Identificados

1. **Confusão com componentes**
   - Tentativa de usar `SpotLight` (não existe)
   - Solução: usar `Light` com propriedade `enabled`

2. **Controle via Grab (Select)**
   - Funciona, mas não era o comportamento desejado
   - Liga ao pegar, desliga ao soltar

3. **Controle via Trigger mantido**
   - Luz acende enquanto pressiona
   - Apaga ao soltar o botão
   - Não mantém o estado (não é toggle)

### Evolução dos Códigos

#### Versão 1: Controle via Grab
```csharp
using UnityEngine;

public class Lamp : MonoBehaviour
{
    public bool LightStatus = false;
    
    void Start()
    {
        GetComponentInChildren<Light>().enabled = false;
    }
    
    public void GrabLight_On()
    {
        GetComponentInChildren<Light>().enabled = true;
    }

    public void GrabLight_Off()
    {
        GetComponentInChildren<Light>().enabled = false;
    }
}
```

**Configuração no Inspector:**
- XR Grab Interactable → Select Entered → `GrabLight_On()`
- XR Grab Interactable → Select Exited → `GrabLight_Off()`

**Resultado:** ✅ Funciona, mas liga/desliga ao pegar/soltar (não é o objetivo final)

---

#### Versão 2: Controle via Trigger (comportamento mantido)
```csharp
using UnityEngine;

public class Lamp : MonoBehaviour
{
    private Light spotLight;  
    public bool LightStatus = false;
    
    void Start()
    {
        spotLight = GetComponentInChildren<Light>();
        spotLight.enabled = false;
    }
    
    public void Trigger_LightOn()
    {
        spotLight.enabled = true;
    }
    
    public void Trigger_LightOff()
    {
        spotLight.enabled = false;
    }
}
```

**Configuração no Inspector:**
- XR Grab Interactable → Activated → `Trigger_LightOn()`
- XR Grab Interactable → Deactivated → `Trigger_LightOff()`

**Resultado:** ✅ Funciona, mas só acende enquanto segura o Trigger

---

#### Versão 3: Método Switch (criado mas não implementado corretamente)
```csharp
void Switch()
{
    LightStatus = !LightStatus;
    GetComponentInChildren<Light>().enabled = LightStatus;
}
```

**Problema:** O método foi criado, mas não foi conectado ao evento correto
**Solução:** Usar eventos via código com `AddListener()`

---

## Solução Implementada

### Código Final (Toggle via AddListener)

```csharp
using UnityEngine;
using UnityEngine.XR.Interaction.Toolkit;

public class Lamp : MonoBehaviour
{
    private Light spotLight;  
    private bool lightStatus = false;
    private XRGrabInteractable grabInteractable;
    
    void Start()
    {
        spotLight = GetComponentInChildren<Light>();
        spotLight.enabled = false;
        
        grabInteractable = GetComponent<XRGrabInteractable>();
        grabInteractable.activated.AddListener(OnActivated);
    }
    
    private void OnActivated(ActivateEventArgs args)
    {
        // Alterna o estado apenas uma vez por ativação
        lightStatus = !lightStatus;
        spotLight.enabled = lightStatus;
    }
    
    void OnDestroy()
    {
        grabInteractable.activated.RemoveListener(OnActivated);
    }
}
```

### Por Que Funciona?

**Diferença crucial:** Eventos no Inspector vs Eventos via Código

| Aspecto | Via Inspector (Luminária 1) | Via Código (Luminária 2) |
|---------|----------------------------|--------------------------|
| **Como configurar** | Arrastar método no Inspector | `AddListener()` no `Start()` |
| **Visibilidade** | ✅ Visual no Inspector | ❌ Invisível (só no código) |
| **Eventos usados** | Activated + Deactivated | Apenas Activated |
| **Comportamento** | Mantido (liga/desliga) | Toggle (alterna) |
| **Quando dispara** | Ao pressionar E ao soltar | Apenas ao pressionar |

**O segredo do toggle:**
```csharp
lightStatus = !lightStatus;  // Operador de negação
```
- Se estava `false` → vira `true` → luz acende
- Se estava `true` → vira `false` → luz apaga

---

## Entendendo Delegates

### O Que São Delegates?

**Definição:** Um delegate é uma "variável que guarda uma função". É um tipo de dado que pode armazenar referências a métodos.

### Analogia do Botão de Campainha

```
🔘 Botão (Delegate)
   ↓ Pode executar diferentes ações:
   - 🔔 Ding-dong (função 1)
   - 🎵 Música (função 2)
   - 🚨 Alarme (função 3)
```

O botão não sabe qual som vai tocar - ele apenas "chama" a função configurada.

### Exemplo Básico em C#

```csharp
// 1. Declarando o tipo do delegate
public delegate void AcaoSimples();

public class ExemploDelegate : MonoBehaviour
{
    // 2. Criando uma variável do tipo delegate
    AcaoSimples minhaAcao;
    
    void Start()
    {
        // 3. Atribuindo uma função ao delegate
        minhaAcao = DizerOla;
        
        // 4. Executando através do delegate
        minhaAcao(); // Output: "Olá!"
        
        // 5. Mudando para outra função
        minhaAcao = DizerTchau;
        minhaAcao(); // Output: "Tchau!"
    }
    
    void DizerOla() => Debug.Log("Olá!");
    void DizerTchau() => Debug.Log("Tchau!");
}
```

### Delegates no Unity XR

Quando fazemos:
```csharp
grabInteractable.activated.AddListener(OnActivated);
```

Estamos dizendo:
> "Quando o evento `activated` disparar, execute a função `OnActivated`"

O evento `activated` é um **delegate** que pode chamar múltiplas funções:

```csharp
// Múltiplos listeners no mesmo evento
grabInteractable.activated.AddListener(OnActivated);
grabInteractable.activated.AddListener(FazerSom);
grabInteractable.activated.AddListener(MostrarParticulas);

// Todas as 3 funções serão executadas quando o Trigger for pressionado!
```

---

## Formas de Adicionar Listeners

### 1. Via Inspector (UnityEvent)
**Como fazer:**
- No Inspector → XR Grab Interactable → Activated
- Arrastar objeto → Selecionar método

**Vantagens:**
- ✅ Visual e intuitivo
- ✅ Não precisa código
- ✅ Fácil para designers

**Desvantagens:**
- ❌ Precisa configurar manualmente
- ❌ Difícil de gerenciar em massa

---

### 2. Via Código com AddListener() (Delegates)
**Como fazer:**
```csharp
void Start()
{
    grabInteractable.activated.AddListener(OnActivated);
}

private void OnActivated(ActivateEventArgs args)
{
    // seu código
}
```

**Vantagens:**
- ✅ Automático ao adicionar o script
- ✅ Flexível e programático
- ✅ Pode adicionar/remover dinamicamente

**Desvantagens:**
- ❌ Precisa conhecer programação
- ❌ Menos visual

---

### 3. Via Lambda (Função Anônima)
**Como fazer:**
```csharp
void Start()
{
    grabInteractable.activated.AddListener((args) => 
    {
        lightStatus = !lightStatus;
        spotLight.enabled = lightStatus;
        Debug.Log("Luz alternada via Lambda!");
    });
}
```

**Vantagens:**
- ✅ Código compacto
- ✅ Bom para ações simples

**Desvantagens:**
- ❌ Difícil de debugar
- ❌ Não pode remover facilmente
- ❌ Menos legível

---

### 4. Via Action/Func
**Como fazer:**
```csharp
void Start()
{
    System.Action<ActivateEventArgs> minhaAcao = (args) =>
    {
        lightStatus = !lightStatus;
        spotLight.enabled = lightStatus;
    };
    
    grabInteractable.activated.AddListener(minhaAcao);
}
```

**Vantagens:**
- ✅ Pode reutilizar a Action

**Desvantagens:**
- ❌ Sintaxe mais complexa

---

### 5. Via Sistema de Eventos C# (event/EventHandler)
**Como fazer:**
```csharp
// Declarando evento customizado
public class Lamp : MonoBehaviour
{
    public event System.Action<bool> OnLightChanged;
    
    public void Switch()
    {
        lightStatus = !lightStatus;
        OnLightChanged?.Invoke(lightStatus); // Notifica listeners
    }
}

// Em outro script, escutando o evento
public class LightMonitor : MonoBehaviour
{
    void Start()
    {
        Lamp lamp = FindObjectOfType<Lamp>();
        lamp.OnLightChanged += OnLampStateChanged;
    }
    
    private void OnLampStateChanged(bool isOn)
    {
        Debug.Log($"Luz: {(isOn ? "ACESA" : "APAGADA")}");
    }
}
```

**Vantagens:**
- ✅ Desacoplamento total
- ✅ Múltiplos objetos podem escutar
- ✅ Padrão Observer

**Desvantagens:**
- ❌ Mais complexo
- ❌ Precisa gerenciar manualmente

---

### 6. Via UnityEvent Customizado
**Como fazer:**
```csharp
using UnityEngine.Events;

[System.Serializable]
public class LightEvent : UnityEvent<bool> { }

public class Lamp : MonoBehaviour
{
    public LightEvent onLightChanged; // Aparece no Inspector!
    
    public void Switch()
    {
        lightStatus = !lightStatus;
        onLightChanged?.Invoke(lightStatus);
    }
}
```

**Vantagens:**
- ✅ Aparece no Inspector
- ✅ Combina código + visual

**Desvantagens:**
- ❌ Requer criar classe customizada

---

### Comparação de Remoção de Listeners

```csharp
// AddListener - precisa da referência ao método
grabInteractable.activated.AddListener(OnActivated);
grabInteractable.activated.RemoveListener(OnActivated);

// Lambda - ❌ NÃO pode remover!
grabInteractable.activated.AddListener((args) => { });
// Não tem referência para remover

// Evento C# - usa -= ao invés de RemoveListener
lamp.OnLightChanged += OnLampStateChanged;
lamp.OnLightChanged -= OnLampStateChanged;
```

---

### Tabela Comparativa

| Método | Complexidade | Flexibilidade | Visível no Inspector | Quando Usar |
|--------|--------------|---------------|---------------------|-------------|
| **Inspector** | ⭐ Fácil | ⭐⭐ Média | ✅ Sim | Configurações simples |
| **AddListener** | ⭐⭐ Média | ⭐⭐⭐ Alta | ❌ Não | Comportamento automático |
| **Lambda** | ⭐⭐⭐ Média | ⭐⭐ Média | ❌ Não | Ações rápidas |
| **Action/Func** | ⭐⭐⭐ Média | ⭐⭐⭐ Alta | ❌ Não | Reutilização |
| **event** | ⭐⭐⭐⭐ Alta | ⭐⭐⭐⭐ Muito Alta | ❌ Não | Sistemas complexos |
| **UnityEvent Custom** | ⭐⭐⭐ Média | ⭐⭐⭐⭐ Muito Alta | ✅ Sim | Eventos próprios |

---

## Controle via Inspector

### Problema Original

```csharp
public bool lightStatus = false; // ❌ Não sincroniza com a luz
```

Quando você marca/desmarca no Inspector, a variável muda, mas a **luz não responde** porque falta código para reagir à mudança.

---

### Solução 1: Propriedade com `set` (Recomendada)

```csharp
using UnityEngine;
using UnityEngine.XR.Interaction.Toolkit;

public class Lamp : MonoBehaviour
{
    private Light spotLight;
    private XRGrabInteractable grabInteractable;
    
    // Variável privada (backing field)
    [SerializeField] private bool _lightStatus = false;
    
    // Propriedade pública que controla a luz
    public bool LightStatus
    {
        get { return _lightStatus; }
        set 
        { 
            _lightStatus = value;
            UpdateLight(); // ← Chama automaticamente ao mudar
        }
    }
    
    void Start()
    {
        spotLight = GetComponentInChildren<Light>();
        grabInteractable = GetComponent<XRGrabInteractable>();
        grabInteractable.activated.AddListener(OnActivated);
        UpdateLight();
    }
    
    private void OnActivated(ActivateEventArgs args)
    {
        LightStatus = !LightStatus; // Usa a propriedade
    }
    
    private void UpdateLight()
    {
        if (spotLight != null)
            spotLight.enabled = _lightStatus;
    }
    
    // Para funcionar no Inspector durante EDIT MODE
    void OnValidate()
    {
        if (!Application.isPlaying)
        {
            spotLight = GetComponentInChildren<Light>();
            UpdateLight();
        }
    }
    
    void OnDestroy()
    {
        grabInteractable.activated.RemoveListener(OnActivated);
    }
}
```

**Como funciona:**
- Variável privada `_lightStatus` armazena o valor
- Propriedade pública `LightStatus` aparece no Inspector
- Quando você muda no Inspector, o `set` chama `UpdateLight()`
- `OnValidate()` garante que funciona em Edit Mode

---

### Solução 2: Context Menu (Botões no Inspector)

```csharp
using UnityEngine;
using UnityEngine.XR.Interaction.Toolkit;

public class Lamp : MonoBehaviour
{
    private Light spotLight;
    private XRGrabInteractable grabInteractable;
    
    [Header("Estado da Luz")]
    [SerializeField] private bool lightStatus = false;
    
    void Start()
    {
        spotLight = GetComponentInChildren<Light>();
        grabInteractable = GetComponent<XRGrabInteractable>();
        grabInteractable.activated.AddListener(OnActivated);
        UpdateLight();
    }
    
    private void OnActivated(ActivateEventArgs args)
    {
        lightStatus = !lightStatus;
        UpdateLight();
    }
    
    private void UpdateLight()
    {
        if (spotLight != null)
            spotLight.enabled = lightStatus;
    }
    
    // ✨ Botões que aparecem no Inspector
    [ContextMenu("Ligar Luz")]
    private void TurnOn()
    {
        lightStatus = true;
        UpdateLight();
    }
    
    [ContextMenu("Desligar Luz")]
    private void TurnOff()
    {
        lightStatus = false;
        UpdateLight();
    }
    
    [ContextMenu("Alternar Luz")]
    private void Toggle()
    {
        lightStatus = !lightStatus;
        UpdateLight();
    }
    
    void OnDestroy()
    {
        grabInteractable.activated.RemoveListener(OnActivated);
    }
}
```

**Como usar:**
1. Clique com botão direito no componente `Lamp` no Inspector
2. Aparecem: "Ligar Luz", "Desligar Luz", "Alternar Luz"
3. Clique na opção desejada

---

### Solução 3: Custom Editor (Avançado)

```csharp
#if UNITY_EDITOR
using UnityEditor;

[CustomEditor(typeof(Lamp))]
public class LampEditor : Editor
{
    public override void OnInspectorGUI()
    {
        DrawDefaultInspector();
        
        Lamp lamp = (Lamp)target;
        
        EditorGUILayout.Space();
        EditorGUILayout.LabelField("Controle Manual", EditorStyles.boldLabel);
        
        if (GUILayout.Button("Ligar Luz"))
            lamp.SetLight(true);
        
        if (GUILayout.Button("Desligar Luz"))
            lamp.SetLight(false);
        
        if (GUILayout.Button("Alternar Luz"))
            lamp.ToggleLight();
    }
}
#endif

public class Lamp : MonoBehaviour
{
    // ... (código anterior)
    
    public void SetLight(bool state)
    {
        lightStatus = state;
        UpdateLight();
    }
    
    public void ToggleLight()
    {
        lightStatus = !lightStatus;
        UpdateLight();
    }
}
```

**Resultado:** Botões grandes e bonitos no Inspector

---

### Comparação das Soluções

| Solução | Complexidade | Edit Mode | Play Mode | Melhor Para |
|---------|--------------|-----------|-----------|-------------|
| **Propriedade + set** | ⭐⭐ Média | ✅ Sim | ✅ Sim | Controle direto via checkbox |
| **Context Menu** | ⭐ Fácil | ✅ Sim | ✅ Sim | Testes rápidos |
| **Custom Editor** | ⭐⭐⭐ Alta | ✅ Sim | ✅ Sim | Interface profissional |

---

## Métodos Especiais do Unity

### `OnValidate()`

**O que é:** Método especial que executa quando você modifica valores no Inspector durante Edit Mode.

**Quando executa:**
1. ✅ Ao modificar valores no Inspector (Edit Mode)
2. ✅ Ao adicionar o script a um objeto
3. ✅ Ao carregar a cena
4. ❌ **NÃO** executa durante Play Mode (se protegido corretamente)

```csharp
void OnValidate()
{
    // Só executa no Editor, não em runtime
    if (Application.isPlaying) return;
    
    spotLight = GetComponentInChildren<Light>();
    
    if (spotLight != null)
    {
        spotLight.enabled = lightStatus;
    }
}
```

**Por que usar `if (Application.isPlaying) return;`?**

| Situação | Sem proteção | Com proteção |
|----------|--------------|--------------|
| **Edit Mode** | `OnValidate()` executa | `OnValidate()` executa ✅ |
| **Play Mode** | `OnValidate()` executa e **conflita** com `Start()` | `OnValidate()` pula ✅ |

**Benefícios:**
- ✅ Ver mudanças do Inspector em tempo real
- ✅ Preview visual sem dar Play
- ✅ Facilita testes e ajustes

**Cuidados:**
```csharp
void OnValidate()
{
    // ❌ EVITE: operações pesadas
    foreach (var obj in FindObjectsOfType<GameObject>())
    {
        // Muito lento!
    }
    
    // ✅ FAÇA: operações rápidas
    spotLight = GetComponentInChildren<Light>();
    
    // ✅ SEMPRE: proteja contra null
    if (spotLight != null)
    {
        spotLight.enabled = lightStatus;
    }
}
```

---

### `Awake()`

**O que é:** Primeiro método que executa quando um objeto é instanciado.

**Quando usar:**
- Inicializar referências internas
- Configurações que não dependem de outros objetos

```csharp
void Awake()
{
    spotLight = GetComponentInChildren<Light>();
    grabInteractable = GetComponent<XRGrabInteractable>();
}
```

**Ordem de execução:**
```
Awake() → OnEnable() → Start() → FixedUpdate() → Update()
```

---

### `Start()`

**O que é:** Executa uma vez antes do primeiro frame, após todos os `Awake()`.

**Quando usar:**
- Inicialização que depende de outros objetos
- Configurar eventos e listeners

```csharp
void Start()
{
    spotLight.enabled = false;
    grabInteractable.activated.AddListener(OnActivated);
}
```

**Diferença de `Awake()`:**
- `Awake()` → executa mesmo se objeto desativado
- `Start()` → só executa se objeto ativo

---

### `Update()`

**O que é:** Executa a cada frame.

**Quando usar:**
- Lógica contínua (movimento, input, etc.)

**Cuidado:** Evite operações pesadas!

```csharp
void Update()
{
    // ❌ EVITE: checar input aqui para toggle
    if (Input.GetButtonDown("Fire1"))
    {
        lightStatus = !lightStatus;
    }
    
    // ✅ PREFIRA: usar eventos do XR Interaction Toolkit
}
```

---

### `OnDestroy()`

**O que é:** Executa quando o objeto é destruído.

**Quando usar:**
- Limpar listeners
- Liberar recursos
- Salvar dados

```csharp
void OnDestroy()
{
    // Sempre remover listeners para evitar memory leaks
    if (grabInteractable != null)
    {
        grabInteractable.activated.RemoveListener(OnActivated);
    }
}
```

**Importante:** Sempre remover listeners adicionados com `AddListener()`!

---

### Tabela de Métodos do Unity

| Método | Quando Executa | Frequência | Uso Principal |
|--------|----------------|------------|---------------|
| **`Awake()`** | Ao instanciar | Uma vez | Referências internas |
| **`OnEnable()`** | Ao ativar objeto | Sempre que ativa | Setup temporário |
| **`Start()`** | Antes do 1º frame | Uma vez | Inicialização |
| **`OnValidate()`** | Ao editar no Inspector | A cada mudança | Preview visual |
| **`Update()`** | Cada frame | Contínuo | Lógica de gameplay |
| **`FixedUpdate()`** | Cada physics step | Contínuo (fixo) | Física |
| **`LateUpdate()`** | Após Update() | Contínuo | Câmera, follow |
| **`OnDestroy()`** | Ao destruir | Uma vez | Limpeza |

---

### Ordem de Execução Completa

```
1. Awake()
   ↓
2. OnEnable()
   ↓
3. Start()
   ↓
4. FixedUpdate() (física - 50x/segundo)
   ↓
5. Update() (cada frame - varia)
   ↓
6. LateUpdate() (após Update)
   ↓
7. OnDisable() (ao desativar)
   ↓
8. OnDestroy() (ao destruir)
```

**Loop de Game:**
```
Start() (uma vez)
   ↓
   ┌──────────────┐
   │ FixedUpdate  │ ← Física (fixo)
   ├──────────────┤
   │ Update       │ ← Lógica (variável)
   ├──────────────┤
   │ LateUpdate   │ ← Após Update
   └──────────────┘
        ↓ (repete)
```

---

## Código Final Completo

### Versão Recomendada (com Controle no Inspector)

```csharp
using UnityEngine;
using UnityEngine.XR.Interaction.Toolkit;

/// <summary>
/// Controla uma luminária interativa em XR
/// Funciona como interruptor: apertar Trigger alterna luz
/// </summary>
public class Lamp : MonoBehaviour
{
    // ========================================
    // CAMPOS PRIVADOS
    // ========================================
    private Light spotLight;
    private XRGrabInteractable grabInteractable;
    
    // Backing field para a propriedade
    [SerializeField] private bool _lightStatus = false;
    
    // ========================================
    // PROPRIEDADE PÚBLICA
    // ========================================
    /// <summary>
    /// Status da luz - visível no Inspector
    /// Atualiza a luz automaticamente ao mudar
    /// </summary>
    public bool LightStatus
    {
        get => _lightStatus;
        set 
        { 
            _lightStatus = value;
            UpdateLight();
        }
    }
    
    // ========================================
    // INICIALIZAÇÃO
    // ========================================
    void Start()
    {
        // Busca referências
        spotLight = GetComponentInChildren<Light>();
        grabInteractable = GetComponent<XRGrabInteractable>();
        
        // Adiciona listener ao evento de ativação
        grabInteractable.activated.AddListener(OnActivated);
        
        // Aplica estado inicial
        UpdateLight();
    }
    
    // ========================================
    // LÓGICA DO TOGGLE
    // ========================================
    /// <summary>
    /// Chamado quando o Trigger é pressionado
    /// Alterna o estado da luz
    /// </summary>
    private void OnActivated(ActivateEventArgs args)
    {
        LightStatus = !LightStatus;
        Debug.Log($"[Lamp] Luz {(LightStatus ? "ACESA" : "APAGADA")}");
    }
    
    /// <summary>
    /// Atualiza o estado físico da luz
    /// </summary>
    private void UpdateLight()
    {
        if (spotLight != null)
        {
            spotLight.enabled = _lightStatus;
        }
    }
    
    // ========================================
    // PREVIEW NO EDITOR
    // ========================================
    /// <summary>
    /// Permite ver mudanças no Inspector em tempo real
    /// </summary>
    void OnValidate()
    {
        // Não interfere durante o jogo
        if (Application.isPlaying) return;
        
        // Atualiza referência
        spotLight = GetComponentInChildren<Light>();
        
        // Aplica estado visual
        UpdateLight();
    }
    
    // ========================================
    // LIMPEZA
    // ========================================
    /// <summary>
    /// Remove listeners ao destruir o objeto
    /// Evita memory leaks
    /// </summary>
    void OnDestroy()
    {
        if (grabInteractable != null)
        {
            grabInteractable.activated.RemoveListener(OnActivated);
        }
    }
    
    // ========================================
    // MÉTODOS AUXILIARES (OPCIONAL)
    // ========================================
    
    [ContextMenu("Ligar Luz")]
    private void TurnOn()
    {
        LightStatus = true;
    }
    
    [ContextMenu("Desligar Luz")]
    private void TurnOff()
    {
        LightStatus = false;
    }
    
    [ContextMenu("Alternar Luz")]
    private void Toggle()
    {
        LightStatus = !LightStatus;
    }
}
```

---

## Resumo Final

### O Que Aprendemos

1. **Eventos no Unity XR**
   - Via Inspector: visual, fácil, mas manual
   - Via código: automático, flexível, programático

2. **Delegates**
   - Variáveis que guardam funções
   - Permitem callbacks e eventos
   - Base do sistema de eventos do Unity

3. **Controle no Inspector**
   - Propriedades com `get/set` para controle direto
   - `OnValidate()` para preview em Edit Mode
   - Context Menu para testes rápidos

4. **Métodos do Unity**
   - `Awake()` → referências internas
   - `Start()` → inicialização e eventos
   - `Update()` → lógica contínua (evitar se possível)
   - `OnValidate()` → preview no Editor
   - `OnDestroy()` → limpeza

### Fluxo Completo do Toggle

```
1. Usuário pega a luminária (Grab)
   ↓
2. Pressiona o Trigger no controle VR
   ↓
3. XR Grab Interactable detecta input
   ↓
4. Evento activated dispara
   ↓
5. AddListener chama OnActivated()
   ↓
6. OnActivated() inverte lightStatus
   ↓
7. Propriedade LightStatus chama set
   ↓
8. set chama UpdateLight()
   ↓
9. UpdateLight() muda spotLight.enabled
   ↓
10. Luz acende/apaga visualmente
```

### Diagrama de Estados

```
       [APAGADA]
          |
    pressiona Trigger
          |
          ↓
       [ACESA]
          |
    pressiona Trigger
          |
          ↓
       [APAGADA]
          |
         ...
```

---

## Referências

- [Unity XR Interaction Toolkit Documentation](https://docs.unity3d.com/Packages/com.unity.xr.interaction.toolkit@latest)
- [C# Delegates and Events](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/delegates/)
- [Unity Execution Order](https://docs.unity3d.com/Manual/ExecutionOrder.html)
- [UnityEvent Documentation](https://docs.unity3d.com/ScriptReference/Events.UnityEvent.html)

---

**Tags:** `#Unity` `#XR` `#VR` `#CSharp` `#Delegates` `#XRInteractionToolkit` `#GameDevelopment`

**Última atualização:** 03/02/2026
