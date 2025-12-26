# Unity XR Interaction Toolkit - Guia de Estudo


Copilot chat link:
https://copilot.microsoft.com/shares/vo4Nj7WpDg7tBoykvXhsT

## Introdução ao XR Interaction Toolkit

O XR Interaction Toolkit é instalado como um pacote através do Unity Package Manager. Frequentemente vem com assets de exemplo, incluindo mapas de ações de entrada e vinculações de controles, para ajudar desenvolvedores a começar rapidamente.

### Configuração Básica

Desenvolvedores configuram o toolkit adicionando componentes como:
- **XR Origin**
- **Interactors** (ex: XR Ray Interactor, XR Direct Interactor)
- **Interactables** (ex: XR Grab Interactable, XR Simple Interactable)

---

## Conceitos Fundamentais

### 🔹 Interactor (Agente Ativo)

**Definição:** Quem faz a ação

**Exemplos:**
- XR Ray Interactor
- XR Direct Interactor

**Função:** Detectar e enviar eventos de interação

**Analogia:** Sua mão virtual ou controle que age no mundo VR

### 🔹 Interactable (Objeto Reativo)

**Definição:** Quem recebe a ação

**Exemplos:**
- XR Grab Interactable
- XR Simple Interactable

**Função:** Definir o comportamento do objeto quando interagido

**Analogia:** Xícara, botão ou qualquer objeto que reage à interação

### Resumo da Diferença

| Componente | Papel | Exemplo Prático |
|------------|-------|-----------------|
| **Interactor** | Agente ativo que realiza ações | Line tracing, mão virtual, laser pointer |
| **Interactable** | Objeto que sofre/reage à ação | Objeto que muda de cor, toca áudio, ou pode ser pego |

---

## Exemplo Prático: Cena Simples VR

### Componentes da Cena

1. **XR Origin** → Representa o player VR
2. **XR Ray Interactor** → Controle direito (raio de interação)
3. **XR Grab Interactable** → Esfera na cena

### Fluxo de Interação

1. Você aponta o raio para a esfera
2. Seleciona a esfera
3. Pega o objeto

---

## Rastreamento: Mundo Real → Mundo Virtual

### 🖐️ Mão Real → Mão Virtual

**Como funciona:**
- Rastreada pelo hardware (headset VR, câmeras, sensores)
- **No Unity:** GameObject dentro do XR Origin com:
  - Componente XR Controller
  - Modelo 3D da mão
  - Interactors anexados

### 🎮 Controle Físico → Controle Virtual

**Como funciona:**
- Controle envia posição, rotação e entrada de botões
- **No Unity:** XR Controller GameObject com:
  - XR Ray Interactor ou XR Direct Interactor
  - Modelo 3D do controle aparece na cena

### Fluxo Completo do Rastreamento

```
Mão/Controle Real (hardware)
         ↓ (rastreamento)
Mão/Controle Virtual (GameObject no Unity)
         ↓ (possui)
Interactor (agente que age)
         ↓ (interage com)
Interactable (objeto que responde)
```

---

## Resumo Geral

### Hierarquia de Componentes

1. **Hardware:** Mão real ou controle físico
2. **Rastreamento:** Sistema captura posição e rotação
3. **Representação Virtual:** GameObject no Unity (mão/controle virtual)
4. **Interactor:** Componente que permite ação (anexado ao GameObject)
5. **Interactable:** Componente em objetos da cena que respondem às ações

### Regra de Ouro

- **Interactor = quem age** (seu controle, mão virtual)
- **Interactable = quem reage** (objetos na cena VR)

---

## Notas para Estudo

- O XR Origin é o ponto de referência para todo o sistema VR
- Um GameObject pode ter múltiplos Interactors
- Interactables definem comportamentos específicos (pegar, empurrar, ativar, etc.)
- O rastreamento é feito automaticamente pelo hardware; Unity apenas recebe os dados

---

**Dica Final:** Pratique criando uma cena simples com os três componentes básicos (XR Origin, Ray Interactor, Grab Interactable) para solidificar o entendimento!