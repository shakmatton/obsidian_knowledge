# Importando Modelos do Blender para Unity XR

## Guia completo para integrar modelos 3D em projetos de realidade virtual

---

## 🎯 Visão Geral

Este guia apresenta o fluxo completo para levar modelos 3D do Blender para dentro de uma cena Unity XR, incluindo preparação, importação, configuração e otimização.

---

## 📋 Índice

1. [Preparar o Modelo no Blender](#1-preparar-o-modelo-no-blender)
2. [Importar para Unity](#2-importar-para-unity)
3. [Ajustar para XR](#3-ajustar-para-xr)
4. [Otimização para XR](#4-otimização-para-xr)
5. [Dicas Extras](#-dicas-extras)

---

## 1. Preparar o Modelo no Blender

### 🧹 Limpeza da Cena

- Remova objetos desnecessários da cena
- Aplique todas as transformações: **Ctrl+A → Apply All Transforms**
- Organize hierarquia de objetos

### 📏 Escala

> ⚠️ **Importante:** Unity usa **metros** como unidade padrão

- Ajuste o modelo para estar na escala correta
- 1 unidade no Blender = 1 metro no Unity
- Verifique dimensões antes de exportar

### 🎨 Materiais e Texturas

- Salve texturas em pastas organizadas
- Use formatos compatíveis:
  - **PNG** (com transparência)
  - **JPG** (sem transparência)
- Nomeie arquivos de forma descritiva

### 📦 Exportação FBX

**Caminho:** File → Export → FBX

#### Configurações recomendadas:

| Configuração | Valor | Motivo |
|-------------|-------|--------|
| **Apply Transform** | ✅ Ativado | Aplica escala/rotação/posição |
| **Selected Objects** | ✅ Apenas Mesh | Evita exportar câmeras/luzes |
| **Forward** | `-Z Forward` | Compatibilidade com Unity |
| **Up** | `Y Up` | Padrão Unity |

```
Exemplo de configuração:
☑ Apply Transform
☑ Selected Objects → Mesh
☐ Include → Cameras (desmarcar)
☐ Include → Lights (desmarcar)
Forward: -Z Forward
Up: Y Up
```

---

## 2. Importar para Unity

### 📂 Adicionar ao Projeto

1. Copie o arquivo `.fbx` (ou `.blend`) para a pasta `Assets` do projeto Unity
2. Unity converte automaticamente e gera um **prefab** do modelo
3. Aguarde o processo de importação terminar

### 🔍 Verificação de Materiais

**No Inspector do modelo importado:**

- Verifique se materiais foram importados corretamente
- Confira se texturas estão vinculadas
- Ajuste manualmente caso necessário:
  - Arraste texturas para os slots de material
  - Configure propriedades de Albedo, Normal, Metallic, etc.

### ⚙️ Configurações de Importação

**Clique no modelo na pasta Assets → Inspector:**

```
Model Tab:
- Scale Factor: 1 (ou ajuste conforme necessário)
- Mesh Compression: Off (para qualidade máxima em XR)
- Read/Write Enabled: ✓ (se for usar mesh colliders)

Rig Tab (se tiver animações):
- Animation Type: Generic ou Humanoid

Materials Tab:
- Location: Use Embedded Materials
- Naming: By Base Texture Name
```

---

## 3. Ajustar para XR

### 🎮 Pré-requisitos

Certifique-se de que está configurado:
- ✅ XR Interaction Toolkit instalado
- ✅ XR Origin presente na cena
- ✅ Controllers configurados

### 📍 Posicionar na Cena

1. Arraste o modelo da pasta Assets para a **Hierarchy**
2. Posicione em relação ao **XR Origin** (ponto de referência do usuário)
3. Ajuste posição/rotação/escala no **Transform**

```
Exemplo de posicionamento:
Position: (0, 1, 2) → 2 metros à frente, 1 metro de altura
Rotation: (0, 0, 0)
Scale: (1, 1, 1) → escala real
```

### 🖐️ Adicionar Interatividade

Se o modelo precisar de interação (pegar, mover, botões), adicione:

#### XR Grab Interactable
```
Add Component → XR Grab Interactable

Configurações básicas:
- Movement Type: Kinematic ou Velocity Tracking
- Throw On Detach: ✓ (para arremesso)
- Attach Transform: Configure ponto de "pegada"
```

#### Colliders
```
Add Component → Box Collider (ou Sphere/Mesh Collider)

Ajuste:
- Is Trigger: ☐ (para interação física)
- Size: Ajuste para cobrir o modelo
```

#### Rigidbody (para física realista)
```
Add Component → Rigidbody

Configurações:
- Mass: 1 (ajuste conforme realismo)
- Drag: 0
- Angular Drag: 0.05
- Use Gravity: ✓
- Interpolate: Interpolate (para suavidade)
- Collision Detection: Continuous Dynamic
```

---

## 4. Otimização para XR

### 🎨 Redução de Polígonos

**No Blender (antes de exportar):**

1. Selecione o objeto
2. Add Modifier → **Decimate**
3. Ajuste **Ratio** (comece com 0.5 para 50% menos polígonos)
4. Teste visual antes de aplicar
5. Apply Modifier

**Meta para XR:**
- Mobile VR: < 50k triângulos por objeto
- PC VR: < 100k triângulos por objeto

### 🖼️ Materiais Simples

- Use shader **Standard** ou **URP/Lit** do Unity
- Evite shaders complexos com muitos passes
- Limite texturas a 2048x2048 (1024x1024 para mobile)
- Use atlas de texturas quando possível

### 💡 Bake de Luzes

**Para evitar cálculos pesados em tempo real:**

1. Configure objetos como **Static**
2. Window → Rendering → Lighting Settings
3. Configure **Lightmapping Settings**
4. Clique em **Generate Lighting**

```
Configuração recomendada:
Lightmap Resolution: 20-40
Lightmap Size: 1024 ou 2048
Compress Lightmaps: ✓
Ambient Occlusion: ✓
```

### 📊 LOD (Level of Detail)

Para objetos que aparecem em diferentes distâncias:

1. Crie versões com menos polígonos no Blender
2. No Unity: Add Component → **LOD Group**
3. Arraste modelos para os níveis LOD0, LOD1, LOD2
4. Ajuste distâncias de transição

---

## 📌 Dicas Extras

### Arquivo .blend vs .fbx

| Formato | Vantagens | Desvantagens |
|---------|-----------|--------------|
| **.blend** | Atualiza automaticamente quando editado no Blender | Requer Blender instalado na máquina |
| **.fbx** | Mais estável, independente do Blender | Precisa reexportar ao fazer mudanças |

**✅ Recomendação:** Use **FBX** para fluxo mais estável e profissional

### 🎬 Animações

Se seu modelo tiver animações:

1. Exporte animações junto com o modelo em FBX
2. No Unity: Model → Rig → Animation Type = Generic/Humanoid
3. Configure **Animator Controller**
4. Adicione component **Animator** ao objeto

### 🧪 Testes

**Sempre teste em Play Mode com XR:**

```
Checklist de testes:
☐ Modelo aparece na posição correta
☐ Escala está adequada (não muito grande/pequeno)
☐ Materiais e texturas estão corretos
☐ Interações funcionam (pegar, soltar, arremessar)
☐ Performance está aceitável (FPS estável)
☐ Colliders estão funcionando corretamente
```

### 🔧 Troubleshooting Comum

**Modelo muito grande/pequeno:**
- Ajuste Scale Factor nas configurações de importação
- Ou reexporte do Blender com escala correta

**Materiais rosa/faltando texturas:**
- Reimporte texturas manualmente
- Verifique se estão na pasta Assets
- Configure material manualmente no Inspector

**Modelo não interagível em XR:**
- Adicione XR Grab Interactable
- Verifique se há Collider
- Adicione Rigidbody se necessário

**Performance ruim:**
- Reduza polígonos no Blender
- Comprima texturas
- Use LOD
- Faça bake de iluminação

---

## 🎯 Fluxo Rápido (Resumo)

### No Blender:
1. ✅ Limpar cena
2. ✅ Aplicar transformações
3. ✅ Ajustar escala (metros)
4. ✅ Exportar como FBX (-Z Forward, Y Up)

### No Unity:
1. ✅ Copiar FBX para pasta Assets
2. ✅ Verificar importação de materiais
3. ✅ Arrastar para cena (posicionar relativo ao XR Origin)
4. ✅ Adicionar XR Grab Interactable + Collider + Rigidbody
5. ✅ Otimizar (reduzir polígonos, bake luzes)
6. ✅ Testar em Play Mode

---

## 📚 Recursos Adicionais

**Documentação Oficial:**
- [Unity XR Interaction Toolkit](https://docs.unity3d.com/Packages/com.unity.xr.interaction.toolkit@latest)
- [Unity Manual - Importing from Blender](https://docs.unity3d.com/Manual/HOWTO-ImportObjectBlender.html)

**Formatos de Exportação Suportados:**
- FBX (recomendado)
- OBJ (sem animações)
- GLTF/GLB (alternativa moderna)
- .blend (direto, requer Blender)

---

*Guia criado para facilitar a integração de modelos 3D do Blender em projetos Unity XR*
