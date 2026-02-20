# Exportar Blender → Unity com Texturas Alpha (FBX)

**Guia completo baseado em solução testada e aprovada**

---

## 📋 Contexto do Problema

Ao exportar modelos do Blender para o Unity usando `.fbx`, as texturas com transparência (alpha) **não vinham junto** com o modelo. Os objetos apareciam com cores básicas, mas sem as texturas aplicadas.

### Por que isso acontece?

- O formato FBX **não embute texturas automaticamente** da forma que o Unity espera
- Texturas complexas feitas com **Shader Nodes** não são reconhecidas pelo Unity
- O Unity precisa de texturas **simplificadas** conectadas diretamente ao material

---

## ✅ Solução que Funciona

### Pré-requisitos

- Blender 2.8+ (com Shader Nodes)
- Unity com **URP (Universal Render Pipeline)** instalado
- Textura PNG com **transparência real** (canal alpha RGBA)

---

## 🔧 PARTE 1: Preparar no Blender

### 1.1 Verificar se a textura tem transparência

**Método rápido (Linux/Mac):**

```bash
file sua_textura.png
```

- Se mostrar **"RGBA"** → tem transparência ✅
- Se mostrar **"RGB"** → NÃO tem transparência ❌

**Método visual (GIMP/Photoshop):**

- Abra a imagem
- Se o fundo mostrar **tabuleiro xadrez** = transparente ✅
- Se mostrar cor sólida = opaco ❌

---

### 1.2 Corrigir orientação da textura (se necessário)

Se a textura estiver **ao contrário** ou **invertida**:

**Opção A - Editar a imagem:**

- Abra no editor de imagem (GIMP, Photoshop, etc.)
- Espelhe/gire até ficar na orientação correta
- Salve como PNG novo

**Opção B - Ajustar UV no Blender:**

1. Selecione o objeto → `Tab` (Edit Mode)
2. UV Editing → Selecione todas as faces (`A`)
3. No UV Editor:
    - `S` → `-1` → `Enter` (espelha horizontal)
    - Ou `R` → `180` → `Enter` (gira 180°)

---

### 1.3 Simplificar os Shader Nodes

**⚠️ PONTO CRÍTICO - O "Pulo do Gato" #1**

O Unity **NÃO lê** nodes complexos do Blender. Você precisa simplificar!

1. Vá na aba **Shading**
    
2. Selecione o material com a textura alpha
    
3. **Delete TODOS os nodes** exceto:
    
    - Principled BSDF
    - Material Output
4. Adicione apenas: `Shift + A` → Texture → **Image Texture**
    
5. Abra sua textura PNG neste node
    
6. **Conecte assim:**
    

```
[Image Texture] ─────────→ [Principled BSDF] ─→ [Material Output]
     │                              │
     ├─ Color ──→ Base Color        │
     └─ Alpha ──→ Alpha ────────────┘
```

7. **Configure o Principled BSDF:**
    - **Blend Mode:** Alpha Blend (ou Alpha Clip)
    - **Show Backface:** ✅ (para ver dos dois lados)

**Exemplo de configuração final:**

- 1 node Image Texture (com sua textura)
- 1 node Principled BSDF
- 1 node Material Output
- **Total: 3 nodes apenas!**

---

### 1.4 Aplicar Transforms

**⚠️ CRÍTICO - Não pule este passo!**

```
Selecione TODOS os objetos → Ctrl + A → All Transforms
```

Isso garante que:

- Escala está aplicada (não será 0.01 ou 100 no Unity)
- Rotação está zerada
- Location está correta
- **O pivô/origin será respeitado no Unity**

---

### 1.5 Exportar FBX

```
File → Export → FBX (.fbx)
```

**Configurações essenciais:**

#### Aba "Include"

- ✅ Selected Objects (ou marque os que quer)
- ✅ Limit to: Selected Objects

#### Aba "Transform"

- ✅ Apply Scalings: **FBX All**
- ✅ Forward: **-Z Forward**
- ✅ Up: **Y Up**
- ✅ Apply Unit
- ✅ Apply Transform

#### Aba "Geometry"

- Smoothing: **Face**
- Export Subdivision Surface: Off

#### Aba "Armature" (se não usar rig)

- ❌ Add Leaf Bones (desmarque)

#### ⚠️ **IMPORTANTE - Path Mode:**

- **Path Mode: Copy**
- **Clique no ícone 📁 ao lado**
- ✅ **Embed Textures** (se disponível)

**Nota:** Mesmo com "Embed Textures" marcado, você ainda precisará copiar as texturas manualmente para o Unity (bug conhecido do Blender-Unity).

---

## 🎮 PARTE 2: Importar no Unity (URP)

### 2.1 Preparar arquivos

1. **Copie manualmente** sua textura PNG para:
    
    ```
    Assets/Textures/
    ```
    
2. **Arraste o FBX** para:
    
    ```
    Assets/Models/
    ```
    

---

### 2.2 Configurar a textura PNG

1. Selecione o PNG em `Assets/Textures/`
2. No **Inspector** (painel direito):

```
Texture Type: Default (ou Sprite 2D)
   
Advanced ▼
  ✅ Alpha Source: Input Texture Alpha
  ✅ Alpha is Transparency
  
[Apply]
```

---

### 2.3 Extrair Materials

**⚠️ PONTO CRÍTICO - O "Pulo do Gato" #2**

1. Na janela **Project**, clique **UMA VEZ** no arquivo `.fbx`
2. No **Inspector** (direita), clique na aba **Materials**
3. Clique em **"Extract Materials..."**
4. Escolha/crie uma pasta (ex: `Assets/Materials/`)
5. Clique **"Select Folder"**

Agora você terá arquivos `.mat` separados na pasta escolhida.

---

### 2.4 Configurar Material (URP)

**⚠️ DESCOBERTA IMPORTANTE - Interface diferente!**

Se você usa **URP (Universal Render Pipeline)**, a interface é diferente do Built-in:

|Built-in Pipeline|URP|
|---|---|
|Rendering Mode|**Surface Type**|
|Albedo|**Base Map**|
|Alpha Cutoff|Threshold (com Alpha Clipping)|

#### Passo a passo:

1. Abra a pasta onde extraiu os materiais
2. **Clique no arquivo `.mat`** do seu objeto
3. No **Inspector**, configure:

```
┌─────────────────────────────────────┐
│ Shader: Universal Render Pipeline/Lit│
│                                     │
│ ▼ Surface Options                   │
│   Workflow Mode: Metallic           │
│   Surface Type: Transparent  ◄─────┐│ ✅ ESSENCIAL! (NÃO use Opaque)
│   Blending Mode: Alpha       ◄─────┤│ ✅ ESSENCIAL!
│   Render Face: Front         ◄─────┘│ ✅ IMPORTANTE! (não Back)
│   Alpha Clipping: □                 │ (deixe desmarcado para alpha suave)
│                                     │
│ ▼ Surface Inputs                    │
│   ⊙ Base Map     [____] 🎨 ◄────────┐│ ✅ ARRASTE SUA TEXTURA PNG AQUI!
│     Metallic Map                    │
│     Smoothness    ▓░░░  0.5         │
└─────────────────────────────────────┘
```

#### Como aplicar a textura no Base Map:

**Método 1 - Arrastar (recomendado):**

1. Abra `Assets/Textures/` na janela Project
2. **Arraste** `sua_textura.png` para o **quadradinho branco** ao lado de "Base Map"

**Método 2 - Selecionar:**

1. Clique no **círculo** (⊙) antes de "Base Map"
2. Selecione sua textura na lista

---

### 2.4.1 Configuração Validada em Uso Real

**⚠️ CONFIGURAÇÃO TESTADA E APROVADA:**

Após múltiplos testes, a configuração que **realmente funciona** é:

```
Material com transparência (alpha):
├─ Surface Type: Transparent      ← NÃO use Opaque!
├─ Blending Mode: Alpha
├─ Render Face: Front             ← Crítico para renderização correta
├─ Alpha Clipping: □ desmarcado   ← Para transparência suave
└─ Base Map: sua_textura.png      ← Arraste o PNG aqui

Material sólido (sem alpha):
├─ Surface Type: Opaque           ← Use Opaque para cores sólidas
├─ Base Color: Escolha a cor
└─ Base Map: [none]
```

**Erros comuns que NÃO funcionam:**

- ❌ Usar Surface Type "Opaque" para materiais com alpha
- ❌ Esquecer de arrastar a textura PNG para o Base Map
- ❌ Não configurar "Alpha is Transparency" no PNG
- ❌ Deixar Render Face em "Back" (pode causar problemas de visualização)

---

### 2.5 Configurações opcionais

#### Alpha Clipping (bordas duras vs suaves)

- **Desmarcado** = Transparência **suave** (gradiente)
- **Marcado** = Transparência **cortada** (sem gradiente)

Se marcar **Alpha Clipping**:

```
Threshold: 0.5  ← Ajuste entre 0.0 e 1.0
```

- Valores menores = mais área transparente
- Valores maiores = menos área transparente

---

## 🎯 Checklist Completo

### No Blender:

- [ ] Textura PNG tem transparência (RGBA)
- [ ] Textura está na orientação correta
- [ ] Nodes simplificados (só Image Texture + Principled BSDF)
- [ ] Alpha conectado ao Principled BSDF
- [ ] Blend Mode: Alpha Blend no material
- [ ] Aplicou transforms: `Ctrl + A → All Transforms`
- [ ] Exportou FBX com "Apply Transform" e "Apply Scalings: FBX All"
- [ ] Path Mode: Copy + tentativa de Embed Textures

### No Unity:

- [ ] Copiou PNG manualmente para `Assets/Textures/`
- [ ] Importou FBX para `Assets/Models/`
- [ ] Configurou PNG: Alpha Source + Alpha is Transparency
- [ ] Extraiu Materials do FBX (Materials tab → Extract Materials)
- [ ] Abriu o material .mat extraído
- [ ] Surface Type: **Transparent** (NÃO Opaque para alpha!)
- [ ] Blending Mode: **Alpha**
- [ ] Render Face: **Front**
- [ ] Arrastou PNG para **Base Map** (não deixe vazio!)
- [ ] Testou no Scene/Game view

---

## 🔥 Os "Pulos do Gato" (Pontos Críticos)

### 1️⃣ Simplificar Nodes no Blender

**Problema:** Nodes complexos não são exportados.  
**Solução:** Deixar apenas Image Texture → Principled BSDF → Material Output.

### 2️⃣ URP usa "Base Map", não "Albedo"

**Problema:** Procurar "Albedo" no URP e não encontrar.  
**Solução:** No URP, o campo se chama **"Base Map"** (em Surface Inputs).

### 3️⃣ Copiar texturas manualmente

**Problema:** FBX "Embed Textures" não funciona 100%.  
**Solução:** Sempre copiar os PNGs manualmente para `Assets/Textures/`.

### 4️⃣ Aplicar Transforms antes de exportar

**Problema:** Escala/rotação errada no Unity.  
**Solução:** `Ctrl + A → All Transforms` no Blender antes de exportar.

### 5️⃣ Surface Type precisa ser "Transparent"

**Problema:** Material importa como opaco.  
**Solução:** Mudar manualmente para "Transparent" no Unity (URP).

### 6️⃣ Render Face deve estar em "Front"

**Problema:** Transparência não renderiza corretamente ou objeto desaparece de certos ângulos.  
**Solução:** Configurar "Render Face: Front" em Surface Options (URP).

---

## 🐛 Troubleshooting

### Textura não aparece no Unity

- ✅ Verificou se o PNG está em `Assets/Textures/`?
- ✅ Extraiu os Materials do FBX?
- ✅ Arrastou a textura para o Base Map?

### Transparência não funciona

- ✅ PNG tem canal alpha (RGBA)?
- ✅ Surface Type está em "Transparent"?
- ✅ Blending Mode está em "Alpha"?
- ✅ Alpha is Transparency marcado no PNG?

### Modelo aparece com escala errada

- ✅ Aplicou `Ctrl + A → All Transforms` no Blender?
- ✅ Apply Scalings: FBX All na exportação?

### Textura aparece espelhada/invertida

- ✅ Corrigiu a orientação no Blender antes de exportar?
- ✅ Ajustou UV Mapping se necessário?

### Material aparece COR DE ROSA (magenta)

- ✅ Shader está correto? Mude para "Universal Render Pipeline/Lit"
- ✅ Extraiu os Materials do FBX? (não pode estar embedded)
- ✅ Se veio de projeto antigo, recrie o material do zero

### Materiais não são editáveis

- ✅ Você PRECISA extrair materials primeiro!
- ✅ Clique no FBX → Materials tab → Extract Materials
- ✅ Depois edite os arquivos .mat extraídos

---

## 📚 Referências Rápidas

### Atalhos Blender

- `Ctrl + A` → Apply Transforms
- `Shift + A` → Add Node
- `Tab` → Edit Mode
- `A` → Select All
- `U` → UV Unwrap Menu

### Formatos de arquivo

- **.fbx** → Funciona, mas sem embed confiável de texturas
- **.glb** → Melhor para embed, mas pode ter problemas no Unity
- **.png** → Use para texturas com alpha (não JPG!)

### Shader differences

|Built-in|URP|HDRP|
|---|---|---|
|Standard|Lit|Lit|
|Albedo|Base Map|Base Map|
|Rendering Mode|Surface Type|Surface Type|

---

## ✅ Resultado Final

Seguindo este guia, você terá:

- ✅ Modelo 3D importado corretamente
- ✅ Texturas com transparência funcionando
- ✅ Escala e pivô corretos
- ✅ Materiais configurados no Unity
- ✅ Workflow replicável para futuros modelos

---

**Criado:** Fevereiro 2026  
**Testado em:** Blender 3.x/4.x + Unity 2022+ (URP)  
**Status:** ✅ Solução validada e funcional