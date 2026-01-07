# Guia Completo: XR Affordance System no Unity

## Sumário
- [Introdução](#introdução)
- [O Que É o XR Affordance System](#o-que-é-o-xr-affordance-system)
- [Componentes Principais](#componentes-principais)
- [Problema Inicial](#problema-inicial)
- [Solução Passo a Passo](#solução-passo-a-passo)
- [Código Final](#código-final)
- [Troubleshooting](#troubleshooting)
- [Conceitos Avançados](#conceitos-avançados)
- [Recursos Adicionais](#recursos-adicionais)

---

## Introdução

**Objetivo:** Criar um efeito de highlighting (brilho no contorno) em um objeto cilindro em uma cena XR do Unity, usando o XR Affordance System.

**Contexto:** Projeto baseado na DemoScene do Unity XR Interaction Toolkit 3.0.

**Documentação Oficial:** https://docs.unity3d.com/Packages/com.unity.xr.interaction.toolkit@3.0/manual/affordance-system.html

### ⚠️ Aviso Importante
O sistema de Affordance foi marcado como **deprecated** no XR Interaction Toolkit 3.0.0 e será substituído por um novo sistema de feedback de interação no futuro. No entanto, ainda funciona perfeitamente.

---

## O Que É o XR Affordance System

O XR Affordance System é um sistema modular que fornece feedback visual e auditivo baseado nos estados de interação de objetos.

### Fluxo de Informação

```
1. INTERACTABLE (objeto interagível)
         ↓
2. AFFORDANCE STATE PROVIDER (monitora estado)
         ↓
3. AFFORDANCE RECEIVER (aplica efeito visual)
         ↓
4. AFFORDANCE THEME (configuração dos valores)
```

### Estados de Interação

- **Idle**: Objeto em repouso, sem interação
- **Hover**: Controller/mão próximo ao objeto
- **Select**: Objeto sendo segurado/selecionado
- **Activate**: Botão de ativação pressionado

---

## Componentes Principais

### 1. XR Interactable Affordance State Provider

**Função:** Monitora o estado do XR Interactable e dispara eventos.

**Propriedades Principais:**
- `Interactable Source`: Referência ao objeto XR Interactable
- `Event Constraints`: Controla quais eventos serão ignorados
  - ⚠️ **IMPORTANTE:** Certifique-se de que "Ignore Hover Events" está desmarcado!

### 2. Affordance Receivers

Componentes que recebem eventos de mudança de estado e aplicam efeitos:

#### Color Material Property Affordance Receiver
- Anima propriedades de cor do material
- Usado para mudar `_RimColor` (brilho de borda)

**Propriedades:**
- `Affordance State Provider`: Referência ao State Provider
- `Affordance Theme Datum`: Theme com configurações de cor
- `Color Property Name`: Nome da propriedade no shader (ex: `_RimColor`)
- `Replace Idle State Value With Initial Value`: Usa cor inicial do material como idle

#### Float Material Property Affordance Receiver
- Anima valores float (intensidade, escala, etc.)

#### Outros Receivers
- Vector Affordance Receivers (posição, rotação, escala)
- Audio Affordance Receiver (sons de interação)

### 3. Material Property Block Helper

**Função:** Gerencia a atualização eficiente de propriedades do material sem criar instâncias duplicadas.

**Propriedades:**
- `Renderer`: Referência ao MeshRenderer do objeto
- `Material Index`: Índice do material (geralmente 0)

### 4. Affordance Theme

**Tipo:** ScriptableObject que armazena configurações reutilizáveis

**Contém:**
- Valores para cada estado (Idle, Hover, Select, Activate)
- Animation Curve para interpolação
- Start Value e End Value para transições suaves

---

## Problema Inicial

### Estrutura da Hierarquia

```
DemoScene
└── MyVRStick1
    └── Cylinder1 (Prefab)
        └── XR Grab Interactable
        └── MeshRenderer
        └── Script: MyXRGrabInteractableCylinder
```

### Comportamento Existente

O cilindro já tinha um script que mudava a cor:
- **Idle/Solto:** Azul
- **Grab/Segurado:** Verde

### Objetivo

Adicionar um **efeito de brilho no contorno** (highlight) quando o controller passa sobre o objeto (hover), similar aos objetos originais da DemoScene.

---

## Solução Passo a Passo

### Passo 1: Criar GameObject para Affordance System

1. Selecione **Cylinder1** no Hierarchy
2. Clique direito → **Create Empty**
3. Renomeie para **"AffordanceSystem"**

**Hierarquia resultante:**
```
Cylinder1
└── AffordanceSystem (GameObject vazio)
```

---

### Passo 2: Adicionar XR Interactable Affordance State Provider

1. Selecione **AffordanceSystem**
2. **Add Component** → **XR Interactable Affordance State Provider**
3. Configure:
   - **Interactable Source**: Arraste **Cylinder1** (objeto pai)
   - **Event Constraints:**
     - ☐ Ignore Hover Events (DESMARQUE)
     - ☐ Ignore Hover Priority Events (DESMARQUE)

---

### Passo 3: Adicionar Affordance Receiver

Ainda no **AffordanceSystem**:

1. **Add Component** → **Color Material Property Affordance Receiver**
   - Isso adiciona automaticamente o **Material Property Block Helper**

2. **Configure Color Material Property Affordance Receiver:**
   - **Affordance State Provider**: Arraste **AffordanceSystem**
   - **Color Property Name**: `_RimColor`
   - ☑️ **Replace Idle State Value With Initial Value**

3. **Configure Material Property Block Helper:**
   - **Renderer**: Arraste **Cylinder1** (o Unity pega o MeshRenderer automaticamente)

---

### Passo 4: Criar/Usar o Material Correto

#### Opção A: Copiar Material da DemoScene

1. Localize **Interactable Instant Cylinder** → **Visuals**
2. Veja qual material está usando (usa shader "Interactable")
3. Copie e adapte para seu cilindro

#### Opção B: Verificar Shader

O material deve usar o shader **"Shader Graphs/Interactable"** que contém:
- `_BaseColor`: Cor principal do objeto
- `_RimColor`: Cor do brilho na borda

**No seu material:**
```
Surface Inputs
├── _BaseColor: Cor azul (sua cor base)
└── _RimColor: (0, 0, 0, 0) - Transparente por padrão
```

---

### Passo 5: Configurar o Theme

#### Solução Recomendada: Usar o Theme da DemoScene

1. Localize **Interactable Instant Cylinder** → **Highlight Interaction Affordance** → **Material Affordance**
2. No **Color Material Property Affordance Receiver**, encontre o **HighlightInteractionColor**
3. Arraste esse theme para o seu **Color Material Property Affordance Receiver**

#### Valores do HighlightInteractionColor (0-255):

| Estado | Start RGBA | End RGBA |
|--------|------------|----------|
| **Idle** | 230, 230, 25, 25 | 230, 230, 25, 25 |
| **Hover** | 255, 200, 145, 255 | 255, 200, 145, 255 |
| **Selected** | 255, 200, 145, 255 | 145, 200, 255, 255 |
| **Activated** | 230, 230, 230, 200 | 230, 230, 230, 200 |

#### Convertido para Unity (0-1.0):

| Estado | End Value RGBA |
|--------|----------------|
| **Idle** | (0.90, 0.90, 0.10, 0.10) |
| **Hover** | (1.0, 0.78, 0.57, 1.0) |
| **Selected** | (0.57, 0.78, 1.0, 1.0) |
| **Activated** | (0.90, 0.90, 0.90, 0.78) |

---

### Passo 6: Conectar XR Interaction Manager

**⚠️ CRÍTICO:** O Affordance não funciona sem o Interaction Manager conectado!

1. Selecione **Cylinder1**
2. No componente **XR Grab Interactable** (ou seu script customizado)
3. Localize o campo **Interaction Manager**
4. Arraste o **XR Interaction Manager** do Hierarchy

---

### Passo 7: Ajustar o Script Customizado

O script original mudava `.material.color` diretamente, o que interfere com o Affordance.

#### Script Anterior (Problemático):

```csharp
_rend.material.color = Color.green;  // Muda tudo
```

#### Script Corrigido:

```csharp
_rend.material.SetColor(ColorPropertyID, Color.green);  // Muda só _BaseColor
```

---

## Código Final

### MyXRGrabInteractableCylinder.cs

```csharp
using UnityEngine;
using UnityEngine.XR.Interaction.Toolkit;
using UnityEngine.XR.Interaction.Toolkit.Interactables;    

namespace MyScripts
{
    public class MyXRGrabInteractableCylinder : XRGrabInteractable  
    {
        private Color _originalColor;
        private Renderer _rend;
        
        // Usar _BaseColor para o shader "Interactable"
        private static readonly int ColorPropertyID = Shader.PropertyToID("_BaseColor");
        
        protected override void Awake()
        {
            base.Awake();
            
            // Capturar renderer e cor original do objeto logo no início
            _rend = GetComponent<Renderer>();
            if (_rend != null && _rend.material != null)
            {
                // Usar GetColor para pegar a cor da propriedade específica
                _originalColor = _rend.material.GetColor(ColorPropertyID);
            }
        }

        protected override void OnSelectEntered(SelectEnterEventArgs args)
        {
            base.OnSelectEntered(args);

            if (_rend != null && _rend.material != null)
            {
                // Usar SetColor para mudar apenas a cor base, não o RimColor
                _rend.material.SetColor(ColorPropertyID, Color.green);
            }
        }

        protected override void OnSelectExited(SelectExitEventArgs args)
        {
            base.OnSelectExited(args);

            if (_rend != null && _rend.material != null)
            {
                // Restaurar a cor original (azul)
                _rend.material.SetColor(ColorPropertyID, _originalColor);
            }
        }
    }
}
```

### Separação de Responsabilidades

| Propriedade | Controlador | Função |
|-------------|-------------|---------|
| `_BaseColor` | Script MyXRGrabInteractableCylinder | Cor principal (azul ↔ verde) |
| `_RimColor` | Affordance System | Brilho da borda (highlight) |

---

## Estrutura Final no Inspector

### Cylinder1

**Componentes:**
- Transform
- Mesh Filter
- Mesh Renderer (Material: CylinderBlueWithRim)
- Capsule Collider
- XR Grab Interactable
  - Interaction Manager: XR Interaction Manager ✓
- MyXRGrabInteractableCylinder (Script)

### AffordanceSystem (filho de Cylinder1)

**Componentes:**
1. **XR Interactable Affordance State Provider**
   - Interactable Source: Cylinder1
   - Event Constraints:
     - ☐ Ignore Hover Events
     - ☐ Ignore Hover Priority Events
     - ☑️ Ignore Focus Events

2. **Material Property Block Helper**
   - Renderer: Cylinder1 (MeshRenderer)
   - Material Index: 0

3. **Color Material Property Affordance Receiver**
   - Affordance State Provider: AffordanceSystem
   - Affordance Theme Datum: HighlightInteractionColor
   - Color Property Name: `_RimColor`
   - ☑️ Replace Idle State Value With Initial Value

---

## Troubleshooting

### Problema: Nada acontece no hover

**Causas possíveis:**

1. **Interaction Manager não conectado**
   - ✅ Solução: Arraste XR Interaction Manager para o campo no XR Grab Interactable

2. **"Ignore Hover Events" marcado**
   - ✅ Solução: Desmarque no XR Interactable Affordance State Provider

3. **Theme não conectado**
   - ✅ Solução: Verifique se Affordance Theme Datum não está "None"

4. **Color Property Name incorreto**
   - ✅ Solução: Use `_RimColor` (C maiúsculo, case-sensitive!)

5. **Collider faltando ou desativado**
   - ✅ Solução: Adicione Capsule Collider ou Mesh Collider ao objeto

### Problema: Efeito "invertido" (some no hover)

**Causa:** _RimColor estava configurado manualmente como amarelo no material

**Solução:** Configure _RimColor como `(0, 0, 0, 0)` no material. O Affordance controlará a cor.

### Problema: Cor muda inteira em vez de só o brilho

**Causa:** Color Property Name está como `_Color` ou `_BaseColor` em vez de `_RimColor`

**Solução:** Mude para `_RimColor` no Color Material Property Affordance Receiver

### Debug: Verificar se hover está funcionando

Adicione temporariamente no script:

```csharp
protected override void OnHoverEntered(HoverEnterEventArgs args)
{
    base.OnHoverEntered(args);
    Debug.Log("✅ HOVER ENTROU!");
}

protected override void OnHoverExited(HoverExitEventArgs args)
{
    base.OnHoverExited(args);
    Debug.Log("❌ HOVER SAIU!");
}
```

Se as mensagens aparecem no Console, o hover funciona e o problema está no Affordance.

---

## Conceitos Avançados

### Por Que Usar SetColor em Vez de .color?

```csharp
// ❌ Cria instância do material (ineficiente)
_rend.material.color = Color.green;

// ✅ Modifica propriedade específica
_rend.material.SetColor("_BaseColor", Color.green);
```

**Vantagens:**
- Mais performático
- Permite controle granular de propriedades
- Compatível com Material Property Blocks
- Evita conflitos com Affordance System

### Shader.PropertyToID()

```csharp
private static readonly int ColorPropertyID = Shader.PropertyToID("_BaseColor");
```

**Benefícios:**
- Converte string para int (mais rápido)
- `static readonly` = calculado uma vez
- Evita string lookups repetidos

### Material Property Block vs Material Instance

**Material Property Block:**
- Não cria cópias do material
- Mais eficiente para múltiplos objetos
- Usado pelo Affordance System

**Material Instance (`.material`):**
- Cria cópia única para o objeto
- Usado pelo nosso script
- Pode coexistir com Material Property Block se controlarem propriedades diferentes

### Por Que o Theme da Unity Funciona Melhor?

#### Idle não é Transparente

- **Nossa tentativa:** `(0, 0, 0, 0)` - Completamente invisível
- **Unity:** `(0.90, 0.90, 0.10, 0.10)` - Leve brilho amarelado

**Benefício:** Transição mais suave, sempre há um sutil "preparo" visual

#### Cores HDR (High Dynamic Range)

O Unity permite valores RGB > 1.0 para brilho intenso:

```
Hover: (2.5, 2.0, 1.5, 1.0)  // Muito mais brilhante que (1, 1, 0, 1)
```

#### Cores "Sujas" vs Primárias

- **Cores primárias puras:** `(1, 1, 0)` - Amarelo agressivo
- **Cores naturais:** `(1.0, 0.78, 0.57)` - Laranja/salmão suave

**Rim Light funciona melhor com tons quentes e menos saturados.**

### Aumentar Intensidade do Brilho

Para deixar o highlight mais forte:

1. **Use valores HDR no Theme:**
   - Clique na cor no Theme
   - Ative modo HDR ou digite valores > 1.0
   - Exemplo: R=3.0, G=2.5, B=1.5

2. **Ajuste propriedades do Shader (se disponíveis):**
   - `Rim Power`: Controla alcance (3-5 = envolve mais)
   - `Rim Intensity`: Controla força (2-5 = muito forte)

3. **Use Float Affordance Receiver adicional:**
   - Controla `_RimPower` ou `_RimIntensity` dinamicamente
   - Idle: 1.0 → Hover: 4.0

---

## Recursos Adicionais

### Documentação Oficial

- [XR Interaction Toolkit - Affordance System](https://docs.unity3d.com/Packages/com.unity.xr.interaction.toolkit@3.0/manual/affordance-system.html)
- [XR Interactable Affordance State Provider](https://docs.unity3d.com/Packages/com.unity.xr.interaction.toolkit@3.0/manual/xr-interactable-affordance-state-provider.html)

### Estrutura de Referência na DemoScene

```
Interactable Instant Cylinder
├── Visuals (MeshRenderer com material)
├── Cylinder_COL (Mesh Collider)
└── Highlight Interaction Affordance
    └── Material Affordance
        ├── Material Property Block Helper
        ├── Color Material Property Affordance Receiver
        └── Float Material Property Affordance Receiver (opcional)
```

### Tipos de Affordance Receivers Disponíveis

| Receiver | Tipo de Propriedade | Uso Comum |
|----------|---------------------|-----------|
| Color Material Property | Color | Cor do material, rim color, emission |
| Float Material Property | Float | Intensidade, alpha, rim power |
| Vector2 Material Property | Vector2 | UV offset, tiling |
| Vector3 Material Property | Vector3 | Posição, direção |
| Vector4 Material Property | Vector4 | Cor + alpha, dados customizados |
| Audio Affordance | AudioClip | Sons de feedback |

### Dicas de Engenharia Reversa

Sempre que encontrar um efeito interessante no Unity:

1. **Inspecione a hierarquia completa**
2. **Liste todos os componentes** do objeto e filhos
3. **Anote valores de configuração**
4. **Teste modificações incrementais**
5. **Compare com sua implementação**

Isso foi exatamente o que levou à descoberta do **HighlightInteractionColor** correto!

---

## Conclusão

### O Que Aprendemos

1. **XR Affordance System** é modular e baseado em eventos
2. **State Provider** monitora, **Receivers** aplicam efeitos
3. **Material Property Blocks** permitem múltiplos controladores
4. **Separação de responsabilidades** evita conflitos:
   - Script controla `_BaseColor`
   - Affordance controla `_RimColor`
5. **Themes da DemoScene** têm valores otimizados
6. **Interaction Manager** é essencial para detecção de hover
7. **Case-sensitivity** importa (`_RimColor` ≠ `_Rimcolor`)
8. **Event Constraints** podem bloquear eventos indesejadamente

### Comportamento Final Alcançado

| Estado | Cor Base (_BaseColor) | Brilho Borda (_RimColor) |
|--------|----------------------|--------------------------|
| **Idle** | Azul (script) | Leve brilho amarelado |
| **Hover** | Azul (mantém) | Laranja brilhante ✨ |
| **Select** | Verde (script) | Azul claro |
| **Soltar** | Azul (script) | Volta ao idle |

### Próximos Passos Possíveis

- Adicionar **Audio Affordance Receiver** para sons
- Usar **Float Affordance Receiver** para animar escala
- Experimentar com **valores HDR** para brilho mais intenso
- Criar **themes customizados** para diferentes objetos
- Explorar **Animation Curves** para efeitos únicos

---

**Documentação criada em:** 2026-01-07  
**Unity Version:** 2022+ com XR Interaction Toolkit 3.0  
**Status:** Sistema deprecated mas funcional

---

## Apêndice: Hierarquia de Referência Completa

```
DemoScene
└── MyVRStick1
    └── Cylinder1 (Prefab)
        ├── Transform
        ├── Mesh Filter
        ├── Mesh Renderer
        │   └── Material: CylinderBlueWithRim
        │       └── Shader: Shader Graphs/Interactable
        │           ├── _BaseColor: (0, 0, 1, 1) - Azul
        │           └── _RimColor: (0, 0, 0, 0) - Transparente
        ├── Capsule Collider
        ├── XR Grab Interactable
        │   └── Interaction Manager: XR Interaction Manager
        ├── MyXRGrabInteractableCylinder (Script)
        └── AffordanceSystem (GameObject vazio)
            ├── XR Interactable Affordance State Provider
            │   ├── Interactable Source: Cylinder1
            │   └── Event Constraints:
            │       ├── Ignore Hover Events: ☐
            │       ├── Ignore Hover Priority Events: ☐
            │       └── Ignore Focus Events: ☑️
            ├── Material Property Block Helper
            │   ├── Renderer: Cylinder1 (MeshRenderer)
            │   └── Material Index: 0
            └── Color Material Property Affordance Receiver
                ├── Affordance State Provider: AffordanceSystem
                ├── Affordance Theme Datum: HighlightInteractionColor
                ├── Color Property Name: _RimColor
                ├── Replace Idle State Value: ☑️
                └── Material Property Block Helper: (auto-assigned)
```

---

*Fim do documento*