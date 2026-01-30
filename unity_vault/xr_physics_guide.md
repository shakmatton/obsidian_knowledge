# Unity XR: Configurações de Física para Objetos Interativos

## Guia completo sobre Attach Modes e otimização de interações

---

## 📦 4 Modos de Configuração de Attach

### 1. **Attach Transform**

**O que faz:**
- O objeto é "teleportado" diretamente para o ponto de attach transform
- Sem simulação física: fica preso na posição/rotação exata
- Movimento instantâneo ("snap")

**O que não faz:**
- Não considera física, massa ou colisões
- Não há suavidade no movimento

**✅ Ideal para:** Objetos que devem alinhar perfeitamente na mão (chaves, ferramentas específicas)

---

### 2. **Dynamic Attach**

**O que faz:**
- Movimenta o objeto usando forças físicas (Rigidbody + colisores)
- Tenta alinhar ao ponto de attach respeitando colisões
- Sensação realista de "pegar"

**O que não faz:**
- Não garante alinhamento perfeito imediato
- Pode apresentar oscilações ou atraso
- Requer Rigidbody configurado

**✅ Ideal para:** Objetos que precisam parecer fisicamente pegos (caixas, bolas)

---

### 3. **Dynamic Attach Kinematic**

**O que faz:**
- Similar ao Dynamic Attach, mas muda para modo kinematic ao ser pego
- Não é afetado por forças externas enquanto segurado
- Evita "chacoalhadas" e colisões estranhas
- Volta a ser dinâmico ao soltar

**O que não faz:**
- Não reage a física enquanto está na mão
- Menos realista em termos de simulação

**✅ Ideal para:** Objetos que você quer mover sem interferência da física, mas que voltam a ser dinâmicos

---

### 4. **Dynamic Attach Velocity Tracking**

**O que faz:**
- Segue a mão usando tracking de velocidade
- Unity calcula velocidade necessária para acompanhar o movimento
- Ao soltar, objeto mantém velocidade acumulada (arremesso natural)

**O que não faz:**
- Não é um "snap" perfeito
- Pode haver atraso ou suavização
- Pode "escorregar" se mal configurado

**✅ Ideal para:** Interações onde você quer arremessar objetos naturalmente

---

## 📊 Comparação Rápida

| Modo | Física ativa | Alinhamento perfeito | Pode ser lançado | Estabilidade |
|------|-------------|---------------------|------------------|--------------|
| **Attach Transform** | ❌ Não | ✅ Sim | ❌ Não | ✅ Alta |
| **Dynamic Attach** | ✅ Sim | ❌ Não | ✅ Sim | ⚠️ Oscilações |
| **Dynamic Attach Kinematic** | ⚠️ Parcial | ✅ Sim | ❌ Não | ✅ Alta |
| **Dynamic Attach Velocity** | ✅ Sim | ❌ Não | ✅ Sim (natural) | ⚠️ Depende |

---

## 🔧 Resolvendo Problemas de Tremedeira

### Ajustes no Rigidbody

```csharp
// Configure estas propriedades:
Rigidbody.interpolation = RigidbodyInterpolation.Interpolate; // ou Extrapolate
Rigidbody.collisionDetectionMode = CollisionDetectionMode.ContinuousDynamic;
// Ajuste massa e drag para valores razoáveis
```

**Interpolate:** Suaviza atualizações entre frames de física e renderização  
**Collision Detection:** Evita que objetos "saltem" ou atravessem colliders

---

### Configuração do XR Grab Interactable

- **Movement Type:** Teste `Kinematic` para eliminar tremores
- **Attach Ease In Time:** Aumente para 0.1–0.3s (transição suave)
- **Track Position/Rotation:** Ajuste multiplicadores de velocidade

---

### Suavização via Script

```csharp
void LateUpdate() {
    transform.position = Vector3.Lerp(
        transform.position, 
        target.position, 
        Time.deltaTime * 15f
    );
    
    transform.rotation = Quaternion.Slerp(
        transform.rotation, 
        target.rotation, 
        Time.deltaTime * 15f
    );
}
```

---

### Camadas de Colisão

Configure o **Layer Collision Matrix** para ignorar colisões entre "Hand" e "Grabbable"  
→ Reduz jitter causado por colisões desnecessárias

---

### Uso Correto de FixedUpdate

- Manipulações de física devem estar em `FixedUpdate`
- Atualizações em `Update` podem gerar tremores

---

## ⚠️ Warning: "Cannot throw a kinematic Rigidbody"

### Por que aparece?

- Rigidbody kinematic não aceita `velocity` ou `angularVelocity`
- XR Toolkit tenta aplicar velocidade ao soltar para simular arremesso
- Como é kinematic, Unity avisa mas ignora o comando

### Por que ainda funciona para segurar?

- XR Toolkit move por transformação direta (não por física)
- Você consegue arrastar normalmente
- Warning só aparece ao tentar arremessar
- Objeto não mantém velocidade da mão

---

## 🎯 Soluções para Arremesso

### Opção 1: Desativar Kinematic ao Soltar
```csharp
// XR Toolkit faz isso automaticamente com Velocity Tracking
rigidbody.isKinematic = false;
```

### Opção 2: Desativar "Throw On Detach"
- Se não precisa arremessar, elimina o warning
- Objeto solta sem aplicar velocidade

### Opção 3: Usar Dynamic Attach + Velocity Tracking
- Permite segurar com física e lançar com velocidade
- Mais realista, mas requer ajustes finos

---

## 🤔 Kinematic vs Velocity Tracking: Qual Usar?

### Criar direto com Velocity Tracking
- Objeto sempre dinâmico
- Segue mão via cálculo de velocidade
- Herda velocidade ao soltar automaticamente
- **Limitação:** Pode tremer enquanto segurado

**✅ Melhor para:** Objetos focados em arremesso (bolas, pedras, frisbees)

---

### Começar Kinematic e depois mudar
- **Segurar:** `isKinematic = true` (estável, sem tremores)
- **Soltar:** Muda para dinâmico e aplica velocidade
- **Vantagem:** Combina estabilidade ao segurar + realismo ao soltar
- **Custo:** Precisa de troca de estado (passo extra)

**✅ Melhor para:** Objetos que você quer segurar suavemente mas ainda arremessar (ferramentas, armas, cubos)

---

## 📊 Comparação Final: Kinematic vs Velocity Tracking

| Configuração | Segurar Suave | Arremesso Realista | Simplicidade |
|--------------|---------------|-------------------|--------------|
| **Velocity Tracking** | ⚠️ Pode tremer | ✅ Sim | ✅ Simples |
| **Kinematic → Dinâmico** | ✅ Suave | ✅ Sim | ⚠️ Precisa troca |

---

## ✅ Estratégia Prática Resumida

### Para Realismo Físico (arremesso prioritário)
→ Use **Velocity Tracking** + ajustes de velocidade + Interpolation

### Para Estabilidade Visual (segurar ferramenta)
→ Use **Kinematic** ou **Attach Transform**

### Para Meio-Termo
→ Combine **Dynamic Attach** com Attach Ease In Time + suavização por script

### Configuração Ideal Geral
1. ✅ Interpolate + Collision Detection no Rigidbody
2. ✅ Movement Type ajustado conforme necessidade
3. ✅ Attach Ease In Time para suavizar
4. ✅ Evitar colisões desnecessárias com a mão
5. ✅ Scripts de suavização para controle extra

---

## 💡 Decisão Rápida

**Não liga para tremedeira + só quer arremessar?**  
→ Crie com Velocity Tracking

**Quer suavidade ao segurar + poder arremessar?**  
→ Comece Kinematic e mude para dinâmico ao soltar

---

*Documento baseado em conversa sobre Unity XR Interaction Toolkit*
