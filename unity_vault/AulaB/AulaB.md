# 🎮 Material Didático - Aula B: Programação em Unity com C#

## 📋 Contexto e Objetivos

Este material foi desenvolvido para estudantes do ensino médio que estão sendo introduzidos à programação de jogos usando Unity e C#. O foco está em criar uma experiência prática e envolvente, onde os conceitos técnicos são apresentados através de exemplos concretos e divertidos.

---

## 🎯 Objetivos de Aprendizagem

Ao final desta aula, os estudantes serão capazes de:
- Compreender o papel do C# no desenvolvimento de jogos Unity
- Utilizar os métodos `Start()` e `Update()` adequadamente
- Manipular GameObjects através de scripts
- Implementar detecção de colisões
- Criar interfaces simples usando TextMeshPro

---

## 📚 Conteúdo Programático

### **1. Revisão: Fundamentos de C#**

Antes de mergulhar no Unity, é importante relembrar os pilares da programação:

**Variáveis e Tipos de Dados**
```csharp
int pontuacao = 0;
float velocidade = 3.5f;
string nomeJogador = "Herói";
bool estaVivo = true;
```

**Métodos (Funções)**
```csharp
void ExibirMensagem()
{
    Debug.Log("Olá, desenvolvedores!");
}
```

**Estruturas Condicionais**
```csharp
if (pontuacao > 100)
{
    Debug.Log("Você venceu!");
}
else
{
    Debug.Log("Continue tentando!");
}
```

**Laços de Repetição**
```csharp
for (int i = 0; i < 10; i++)
{
    Debug.Log("Contagem: " + i);
}
```

---

### **2. Bibliotecas e Estrutura de Scripts Unity**

No Unity, todo script C# segue uma estrutura padrão:

```csharp
using UnityEngine;
using TMPro;

public class MeuPrimeiroScript : MonoBehaviour
{
    void Start() 
    {
        // Código executado uma vez ao iniciar
    }
    
    void Update() 
    {
        // Código executado a cada frame
    }
}
```

**O que significam essas linhas?**
- `using UnityEngine;` → Importa funcionalidades essenciais do Unity
- `using TMPro;` → Permite trabalhar com textos estilizados
- `MonoBehaviour` → Classe base que conecta o script ao motor do Unity

---

### **3. Métodos Especiais: Start e Update**

Estes são os dois métodos mais importantes para iniciantes:

**Start()** - Executa uma única vez quando o objeto é ativado
```csharp
void Start()
{
    Debug.Log("O jogo iniciou!");
}
```

**Update()** - Executa continuamente a cada quadro (frame)
```csharp
void Update()
{
    // Move o objeto para a direita constantemente
    transform.Translate(Vector3.right * Time.deltaTime);
}
```

> **Nota Importante:** `Time.deltaTime` garante que o movimento seja suave e independente da taxa de frames.

---

### **4. GameObjects e Componentes**

**Conceito fundamental:** Tudo no Unity é um GameObject (cubo, câmera, personagem, luz). Cada GameObject pode ter vários componentes anexados que definem seu comportamento.

**Componentes comuns:**
- Transform → posição, rotação, escala
- Renderer → aparência visual
- Collider → detecção de colisões
- Rigidbody → física realista
- Scripts personalizados → comportamentos únicos

---

### **5. Sistema de Colisões**

O Unity oferece duas formas principais de detecção:

**OnCollisionEnter** - Para objetos físicos
```csharp
void OnCollisionEnter(Collision colisao)
{
    Debug.Log("Bateu em: " + colisao.gameObject.name);
}
```

**OnTriggerEnter** - Para áreas de detecção (Collider marcado como Trigger)
```csharp
void OnTriggerEnter(Collider outro)
{
    if (outro.CompareTag("Moeda"))
    {
        Debug.Log("Moeda coletada!");
        Destroy(outro.gameObject);
    }
}
```

---

### **6. Interface com TextMeshPro**

Para exibir informações na tela de forma profissional:

```csharp
using TMPro;

public class GerenciadorPontos : MonoBehaviour
{
    public TextMeshProUGUI textoInterface;
    private int pontos = 0;
    
    void OnTriggerEnter(Collider item)
    {
        if (item.CompareTag("Item"))
        {
            pontos += 10;
            textoInterface.text = "Pontos: " + pontos;
            Destroy(item.gameObject);
        }
    }
}
```

---

## 🛠️ Exercícios Práticos Guiados

### **Exercício 1: Objeto em Movimento**
**Objetivo:** Criar um cubo que se desloca automaticamente

```csharp
using UnityEngine;

public class MovimentoAutomatico : MonoBehaviour
{
    void Start()
    {
        Debug.Log("Iniciando movimento!");
    }
    
    void Update()
    {
        transform.Translate(Vector3.forward * 2f * Time.deltaTime);
    }
}
```

**Checklist:**
- [ ] Criar um cubo na cena
- [ ] Anexar o script ao cubo
- [ ] Observar o movimento no Game View
- [ ] Experimentar diferentes direções (Vector3.up, Vector3.left)

---

### **Exercício 2: Mudança de Cor Interativa**
**Objetivo:** Alterar a cor de um objeto ao pressionar uma tecla

```csharp
using UnityEngine;

public class TrocaCor : MonoBehaviour
{
    void Update()
    {
        if (Input.GetKeyDown(KeyCode.Space))
        {
            GetComponent<Renderer>().material.color = Color.blue;
        }
    }
}
```

**Desafio:** Faça o objeto alternar entre três cores diferentes!

---

### **Exercício 3: Detecção de Colisão Simples**
**Objetivo:** Registrar quando dois objetos se tocam

```csharp
using UnityEngine;

public class DetectorColisao : MonoBehaviour
{
    void OnCollisionEnter(Collision col)
    {
        Debug.Log("Colisão detectada com: " + col.gameObject.name);
        
        // Opcional: mudar cor ao colidir
        GetComponent<Renderer>().material.color = Color.red;
    }
}
```

**Setup necessário:**
- [ ] Criar dois objetos com Colliders
- [ ] Adicionar Rigidbody em pelo menos um deles
- [ ] Testar a colisão movendo os objetos

---

### **Exercício 4: Sistema de Pontuação**
**Objetivo:** Criar um sistema funcional de coleta e pontos

```csharp
using UnityEngine;
using TMPro;

public class SistemaPontuacao : MonoBehaviour
{
    public TextMeshProUGUI displayPontos;
    private int pontuacao = 0;
    
    void Start()
    {
        AtualizarDisplay();
    }
    
    void OnTriggerEnter(Collider obj)
    {
        if (obj.CompareTag("Coletavel"))
        {
            pontuacao++;
            AtualizarDisplay();
            Destroy(obj.gameObject);
        }
    }
    
    void AtualizarDisplay()
    {
        displayPontos.text = "Pontos: " + pontuacao;
    }
}
```

**Configuração:**
1. Criar UI → TextMeshPro
2. Marcar objetos coletáveis com a tag "Coletavel"
3. Configurar Colliders como Triggers
4. Arrastar o texto para o campo público no Inspector

---

## 🚀 Tópicos Complementares

### **Captura de Entrada (Input)**
```csharp
void Update()
{
    float movimento = Input.GetAxis("Horizontal");
    transform.Translate(Vector3.right * movimento * 5f * Time.deltaTime);
}
```

### **Física com Rigidbody**
```csharp
Rigidbody rb;

void Start()
{
    rb = GetComponent<Rigidbody>();
}

void Update()
{
    if (Input.GetKeyDown(KeyCode.Space))
    {
        rb.AddForce(Vector3.up * 500f);
    }
}
```

### **Boas Práticas de Organização**
- Nomeie scripts com clareza: `ControladorJogador`, `GerenciadorInimigos`
- Use comentários para explicar lógica complexa
- Organize arquivos em pastas (Scripts, Prefabs, Materials)
- Evite código duplicado criando métodos reutilizáveis

---

## 🎲 Projeto Integrador: Jogo de Coleta

**Descrição:** Criar um mini-jogo onde o jogador controla um cubo que coleta esferas enquanto evita obstáculos.

**Requisitos:**
1. **Personagem controlável** com teclas de seta
2. **Objetos coletáveis** que somam pontos
3. **Obstáculos** que encerram o jogo
4. **Interface visual** mostrando pontuação
5. **Feedback no console** para eventos importantes

**Estrutura sugerida:**
```
Scripts/
├── ControladorJogador.cs  // Movimento do personagem
├── ItemColetavel.cs       // Comportamento dos itens
├── GerenciadorJogo.cs     // Pontuação e game over
└── Obstaculo.cs           // Detecção de fim de jogo
```

---

## 📝 Resumo Visual (Referência Rápida)

| Conceito             | Uso                 | Exemplo                        |
| -------------------- | ------------------- | ------------------------------ |
| **Start()**          | Inicialização única | Carregar configurações         |
| **Update()**         | Loop contínuo       | Verificar input, mover objetos |
| **OnCollisionEnter** | Colisão física      | Detectar impacto               |
| **OnTriggerEnter**   | Área de detecção    | Coletar itens                  |
| **Debug.Log**        | Mensagens de teste  | Acompanhar valores             |
| **Time.deltaTime**   | Suavizar movimento  | Multiplicar velocidades        |

---

## ✅ Checklist de Aprendizagem

- [ ] Compreendo a diferença entre Start e Update
- [ ] Sei importar bibliotecas com `using`
- [ ] Consigo manipular GameObjects via script
- [ ] Entendo a diferença entre Collision e Trigger
- [ ] Sei implementar um sistema de pontuação básico
- [ ] Consigo capturar entrada do teclado
- [ ] Compreendo o conceito de componentes
- [ ] Sei usar Debug.Log para testar código

---

## 📚 Recursos Adicionais

### Documentação Oficial
- [Unity Manual](https://docs.unity3d.com/Manual/index.html)
- [C# Scripting Reference](https://docs.unity3d.com/ScriptReference/)
- [TextMeshPro Documentation](https://docs.unity3d.com/Packages/com.unity.textmeshpro@latest)

### Tutoriais Recomendados
- Unity Learn - Beginner Scripting
- Brackeys (YouTube) - Unity Tutorials
- Code Monkey (YouTube) - Game Development

---

## 💡 Dicas para Estudos

1. **Pratique diariamente** - Mesmo que por 15 minutos
2. **Experimente modificar códigos** - Quebre coisas e conserte
3. **Use Debug.Log constantemente** - Entenda o fluxo do programa
4. **Crie projetos pequenos** -Omplete-os antes de iniciar novos
5. **Participe de comunidades** - Unity Forum, Reddit r/Unity3D

---

**Versão:** 1.0  
**Última atualização:** Novembro 2025  
**Público-alvo:** Estudantes de Ensino Médio  
**Pré-requisitos:** Conhecimento básico de lógica de programação

---

Este material foi estruturado para proporcionar uma progressão natural do conhecimento, começando com conceitos fundamentais e avançando para aplicações práticas. Encoraje os estudantes a experimentarem e modificarem os códigos - a melhor forma de aprender programação é praticando!