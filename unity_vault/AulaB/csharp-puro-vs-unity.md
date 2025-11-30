# 🎯 C# Puro vs C# [[AulaB|Unity]] - Guia Completo

## 📋 Introdução

Este guia explica as diferenças entre C# puro (standalone) e C# no Unity, além de mostrar 4 formas diferentes de testar código C# fora do ambiente Unity.

---

## 📌 C# Puro vs C# Unity

### **C# Puro (Standalone)**
- É o C# "tradicional", usado para criar aplicações console, desktop, web, etc.
- **NÃO** depende do Unity
- Roda direto no .NET Runtime
- Usa `Console.WriteLine()` para imprimir mensagens
- Tem um método `Main()` como ponto de entrada

### **C# no Unity**
- É C# adaptado para funcionar dentro do motor Unity
- **Depende** do MonoBehaviour e do ciclo de vida do Unity
- Usa `Debug.Log()` para imprimir mensagens
- Não tem método `Main()` — usa `Start()` e `Update()`
- Tem acesso a classes especiais como `GameObject`, `Transform`, `Vector3`, etc.

---

## 💻 Como Testar C# Puro (4 Opções)

### **Opção 1: Rider (Recomendado para quem já tem)**

**Passo a passo:**

1. **Abrir o Rider**
2. **File** → **New** → **Project**
3. Selecionar **Console Application** (.NET ou .NET Core)
4. Dar um nome (ex: `TestesCSharp`)
5. Clicar em **Create**

Você terá um arquivo `Program.cs` assim:

```csharp
using System;

namespace TestesCSharp
{
    class Program
    {
        static void Main(string[] args)
        {
            Console.WriteLine("Olá, mundo!");
        }
    }
}
```

**Para testar os conceitos da aula:**

```csharp
using System;

namespace TestesCSharp
{
    class Program
    {
        static void Main(string[] args)
        {
            // VARIÁVEIS
            int idade = 16;
            float velocidade = 5.0f;
            string nome = "Jogador";
            
            Console.WriteLine("Nome: " + nome);
            Console.WriteLine("Idade: " + idade);
            Console.WriteLine("Velocidade: " + velocidade);
            
            // CONDICIONAIS
            if (idade >= 18)
            {
                Console.WriteLine("Maior de idade");
            }
            else
            {
                Console.WriteLine("Menor de idade");
            }
            
            // LOOPS
            for (int i = 0; i < 5; i++)
            {
                Console.WriteLine("Número: " + i);
            }
            
            // CHAMANDO UMA FUNÇÃO
            MostrarMensagem();
        }
        
        // FUNÇÃO/MÉTODO
        static void MostrarMensagem()
        {
            Console.WriteLine("Olá, mundo!");
        }
    }
}
```

**Para rodar:** Apertar o botão ▶️ (Run) no Rider.

---

### **Opção 2: Visual Studio Code (Gratuito e Leve)**

**Pré-requisitos:**
- Instalar o [.NET SDK](https://dotnet.microsoft.com/download)
- Instalar a extensão **C# Dev Kit** no VS Code

**Passo a passo:**

1. Abrir o terminal (PowerShell, CMD ou Terminal do VS Code)
2. Navegar até uma pasta onde quer criar o projeto
3. Executar:
```bash
dotnet new console -n TestesCSharp
cd TestesCSharp
code .
```

4. Isso criará um projeto console e abrirá no VS Code
5. Editar o arquivo `Program.cs`
6. Rodar com: `dotnet run` no terminal

**Vantagens do VS Code:**
- Leve e rápido
- Gratuito e open-source
- Multiplataforma (Windows, Mac, Linux)
- Extensível com plugins

---

### **Opção 3: Visual Studio (IDE Completa da Microsoft)**

**Pré-requisitos:**
- Baixar e instalar o [Visual Studio Community](https://visualstudio.microsoft.com/pt-br/downloads/) (versão gratuita)
- Durante a instalação, selecionar o workload **"Desenvolvimento para desktop com .NET"**

**Passo a passo:**

1. **Abrir o Visual Studio**
2. Clicar em **"Criar um projeto"** (ou File → New → Project)
3. Na busca, digitar **"Console"**
4. Selecionar **"Aplicativo de Console"** (Console App) com C#
5. Clicar em **Avançar**
6. Dar um nome ao projeto (ex: `TestesCSharp`)
7. Escolher a localização onde salvar
8. Clicar em **Criar**

O Visual Studio criará automaticamente um arquivo `Program.cs`:

```csharp
// Versão .NET 6+ (mais simples)
Console.WriteLine("Olá, mundo!");

// OU versão tradicional
using System;

namespace TestesCSharp
{
    class Program
    {
        static void Main(string[] args)
        {
            Console.WriteLine("Olá, mundo!");
        }
    }
}
```

**Para adicionar os exemplos da aula:**

```csharp
using System;

namespace TestesCSharp
{
    class Program
    {
        static void Main(string[] args)
        {
            // VARIÁVEIS
            int idade = 16;
            float velocidade = 5.0f;
            string nome = "Jogador";
            
            Console.WriteLine("Nome: " + nome);
            Console.WriteLine("Idade: " + idade);
            Console.WriteLine("Velocidade: " + velocidade);
            
            // CONDICIONAIS
            if (idade >= 18)
            {
                Console.WriteLine("Maior de idade");
            }
            else
            {
                Console.WriteLine("Menor de idade");
            }
            
            // LOOPS
            for (int i = 0; i < 5; i++)
            {
                Console.WriteLine("Número: " + i);
            }
            
            // CHAMANDO UMA FUNÇÃO
            MostrarMensagem();
            
            // Manter console aberto
            Console.ReadKey();
        }
        
        // FUNÇÃO/MÉTODO
        static void MostrarMensagem()
        {
            Console.WriteLine("Olá, mundo!");
        }
    }
}
```

**Para rodar:** 
- Apertar **F5** (com debug) ou **Ctrl+F5** (sem debug)
- Ou clicar no botão ▶️ verde no topo

**Vantagens do Visual Studio:**
- Interface mais completa e intuitiva
- IntelliSense robusto (autocomplete inteligente)
- Debugger visual poderoso
- Ferramenta "profissional" usada na indústria
- Integração excelente com .NET

---

### **Opção 4: Compiladores Online (Mais Rápido para Testes)**

**Não precisa instalar nada!** Use sites como:

- **[dotnetfiddle.net](https://dotnetfiddle.net/)** ⭐ Recomendado
- **[replit.com](https://replit.com/)** (criar um projeto C#)
- **[onlinegdb.com/online_csharp_compiler](https://www.onlinegdb.com/online_csharp_compiler)**

**Vantagens:**
- Zero instalação
- Perfeito para demonstrações rápidas em aula
- Funciona em qualquer dispositivo com navegador
- Ideal para testar snippets pequenos

**Como usar (dotnetfiddle.net):**
1. Acessar o site
2. Colar o código na área de edição
3. Clicar em **"Run"**
4. Ver o resultado no painel à direita

---

## 🔄 Adaptando o Exemplo da Aula para C# Puro

O código original da sua aula usa `Debug.Log()` (Unity). Para C# puro, substitua por `Console.WriteLine()`:

### **Versão Unity** ❌
```csharp
void MostrarMensagem()
{
    Debug.Log("Olá, mundo!");
}
```

### **Versão C# Puro** ✅
```csharp
static void MostrarMensagem()
{
    Console.WriteLine("Olá, mundo!");
}
```

---

## 📚 Diferenças Principais (Tabela Comparativa)

| Aspecto | C# Puro | C# Unity |
|---------|---------|----------|
| **Onde roda** | Console, Desktop, Web | Dentro do Unity Engine |
| **Ponto de entrada** | `Main()` | `Start()` / `Awake()` |
| **Imprimir mensagens** | `Console.WriteLine()` | `Debug.Log()` |
| **Classes base** | Nenhuma obrigatória | `MonoBehaviour` |
| **Acesso a objetos** | Não tem GameObjects | `GameObject`, `Transform`, etc. |
| **Uso** | Apps gerais | Jogos e aplicações Unity |
| **Modificador static** | Obrigatório em `Main()` | Não usado em métodos Unity |
| **Namespace** | Geralmente define um | Pode usar ou não |
| **Bibliotecas** | `using System;` | `using UnityEngine;` |

---

## 🎓 Recomendação para Sua Aula

Para a **Parte 1 (Aquecimento)**, eu sugiro:

1. **Mostrar os conceitos em C# puro primeiro** usando um compilador online (dotnetfiddle.net) ou Visual Studio
   - Os alunos veem o resultado imediatamente
   - Não precisa abrir o Unity ainda
   - Foco total na sintaxe do C#

2. **Depois migrar para o Unity** e mostrar:
   - "Viram aquele `Console.WriteLine()`? No Unity usamos `Debug.Log()`"
   - "Viram aquele `Main()`? No Unity usamos `Start()` e `Update()`"
   - "O resto é **exatamente igual**: variáveis, if/else, loops..."

Isso deixa claro que **C# é C#**, mas cada ambiente tem suas particularidades.

---

## 🛠️ Exemplo Completo para Testar Agora

Cole isso no **dotnetfiddle.net** ou **Visual Studio** e clique em Run:

```csharp
using System;

public class Program
{
    public static void Main()
    {
        Console.WriteLine("=== TESTANDO C# PURO ===\n");
        
        // Variáveis
        int idade = 16;
        float velocidade = 5.0f;
        string nome = "Jogador";
        
        Console.WriteLine("Nome: " + nome);
        Console.WriteLine("Idade: " + idade);
        Console.WriteLine("Velocidade: " + velocidade + "\n");
        
        // Condicionais
        if (idade >= 18)
        {
            Console.WriteLine("Maior de idade\n");
        }
        else
        {
            Console.WriteLine("Menor de idade\n");
        }
        
        // Loops
        Console.WriteLine("Contando de 0 a 4:");
        for (int i = 0; i < 5; i++)
        {
            Console.WriteLine("Número: " + i);
        }
        
        // Função
        Console.WriteLine();
        MostrarMensagem();
    }
    
    static void MostrarMensagem()
    {
        Console.WriteLine("Olá, mundo! Esta é uma função!");
    }
}
```

**Resultado esperado:**
```
=== TESTANDO C# PURO ===

Nome: Jogador
Idade: 16
Velocidade: 5

Menor de idade

Contando de 0 a 4:
Número: 0
Número: 1
Número: 2
Número: 3
Número: 4

Olá, mundo! Esta é uma função!
```

---

## 🆚 Qual Opção Escolher?

| Situação | Melhor Opção |
|----------|--------------|
| **Para aulas demonstrativas rápidas** | Opção 4 (Compiladores Online) |
| **Alunos sem computador potente** | Opção 4 (Online) ou Opção 2 (VS Code) |
| **Curso completo de C#** | Opção 3 (Visual Studio) |
| **Já usa Rider para Unity** | Opção 1 (Rider) |
| **Quer algo leve e rápido** | Opção 2 (VS Code) |
| **Quer experiência profissional completa** | Opção 3 (Visual Studio) |

---

## 💡 Dicas Importantes

### **Por que usar `static` em C# Puro?**
Em C# puro, o método `Main()` e outros métodos chamados diretamente dele precisam ser `static` porque são executados sem criar uma instância da classe. No Unity, não usamos `static` porque o Unity cria instâncias dos scripts automaticamente.

### **Console.ReadKey() no Visual Studio**
Se você executar um programa no Visual Studio e a janela do console fechar imediatamente, adicione `Console.ReadKey();` no final do método `Main()`. Isso mantém a janela aberta até você pressionar uma tecla.

### **Versões do .NET**
- **.NET Framework**: versão antiga, só Windows
- **.NET Core / .NET 5+**: versão moderna, multiplataforma
- Para iniciantes, use a versão mais recente do .NET disponível

---

## 📚 Recursos Adicionais

### Documentação Oficial
- [Documentação C# da Microsoft](https://learn.microsoft.com/pt-br/dotnet/csharp/)
- [Tutorial C# para Iniciantes](https://learn.microsoft.com/pt-br/dotnet/csharp/tour-of-csharp/)
- [.NET Download](https://dotnet.microsoft.com/download)

### Tutoriais Recomendados
- Microsoft Learn - C# Fundamentals
- Curso C# do Programador Br (YouTube)
- Balta.io - Cursos de C#

---

## ✅ Checklist de Verificação

Antes de começar a programar em Unity, certifique-se de que consegue:

- [ ] Criar e executar um programa console simples
- [ ] Declarar variáveis de diferentes tipos
- [ ] Usar estruturas condicionais (if/else)
- [ ] Criar e usar loops (for, while)
- [ ] Criar e chamar métodos/funções
- [ ] Entender a diferença entre `Console.WriteLine()` e `Debug.Log()`
- [ ] Saber que `Main()` equivale a `Start()` no Unity

---

**Versão:** 1.0  
**Última atualização:** Novembro 2025  
**Autor:** Material de Apoio - Aula B Unity  
**Objetivo:** Auxiliar estudantes do ensino médio na transição de C# puro para C# Unity

---

Este documento pode ser usado como material de referência offline para preparação de aulas ou estudo individual.