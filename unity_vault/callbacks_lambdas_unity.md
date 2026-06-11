# Callbacks, Delegates e Funções Lambda em C# / Unity

> Notas de estudo geradas a partir do desenvolvimento do script `Door.cs` em Unity 6.  
> Contexto: investigação sobre como Colliders funcionam no XR Interaction Toolkit.

---

## 1. O Ponto de Partida

Enquanto depurava o objeto `Porta`, foi encontrado no código-fonte do Unity (`XRBaseInteractable.cs`) o seguinte trecho no método `Awake`:

```csharp
protected virtual void Awake()
{
    if (m_Colliders.Count == 0)
    {
        GetComponentsInChildren(m_Colliders);
        m_Colliders.RemoveAll(col => col.isTrigger);
    }
}
```

A linha `m_Colliders.RemoveAll(col => col.isTrigger);` foi o ponto de partida para entender **callbacks**, **delegates** e **lambdas**.

---

## 2. O que é um Callback?

Um **callback** é uma **função passada como argumento para outra função**.

Em vez de passar um valor (`int`, `string`...), você passa um **comportamento** — um bloco de código que será executado dentro da outra função.

### Exemplo prático (criado em `Door.cs`)

No `Start()`, a chamada seria:

```csharp
RemoveAll(teste); // "teste" é o callback — passado sem parênteses
```

No corpo da classe, as três peças necessárias:

```csharp
// 1. A função que será passada como callback.
//    Recebe um int e retorna bool — define o critério de remoção.
private bool teste(int a)
{
    return true;
}
```

```csharp
// 2. O delegate: NÃO é um método — é uma DEFINIÇÃO DE TIPO.
//    Declara o "contrato de assinatura" que qualquer callback
//    passado para RemoveAll deve respeitar:
//    precisa receber um int e retornar um bool.
//    A função "teste" respeita esse contrato, então pode ser passada.
public delegate bool Callback(int i);
```

```csharp
// 3. RemoveAll recebe o callback como parâmetro.
//    Itera a lista e remove todo elemento para o qual
//    o callback retornar true.
public void RemoveAll(Callback callback)
{
    foreach (int i in _list)
    {
        bool result = callback(i); // executa a função passada, usando i como argumento
        if (result == true)
        {
            _list.Remove(i);
        }
    }
}
```

---

## 3. O que é um Delegate?

O `delegate` **não é um método**. É uma **definição de tipo**, como um contrato de assinatura.

Ele declara qual formato uma função deve ter para poder ser passada como callback.

```csharp
public delegate bool Callback(int i);
// Significa: "qualquer função que receba um int e retorne um bool
//             pode ser usada onde um Callback for esperado."
```

> **Erro comum:** pensar que o parâmetro do delegate "contém" a função passada.  
> Na verdade, ele apenas define o **tipo do parâmetro** que a função callback receberá quando for invocada.

---

## 4. O que é uma Função Lambda?

Uma **lambda** é um callback escrito **inline**, sem precisar declarar uma função separada com nome.

### Sintaxe

```
parâmetro => expressão
```

Significa: *"receba este parâmetro e retorne o resultado desta expressão."*

### Comparação direta

```csharp
RemoveAll(teste);       // callback nomeado (função declarada separadamente)
RemoveAll(a => true);   // lambda equivalente (retorna true para qualquer elemento)
```

Ambas fazem a mesma coisa. A lambda é apenas a forma inline.

---

## 5. Fechando o Ciclo: a Linha do Unity

```csharp
m_Colliders.RemoveAll(col => col.isTrigger);
```

**Tradução:** *"Remova da lista todo `col` (collider) para o qual `col.isTrigger` for `true`."*

É exatamente o `RemoveAll(teste)` do exemplo, com duas diferenças:

| | Exemplo didático | Linha do Unity |
|---|---|---|
| Callback | função nomeada `teste` | lambda escrita inline |
| Critério de remoção | `return true` (sempre) | `col.isTrigger` (condicional) |

---

## 6. Atenção: o símbolo `=>` tem dois significados distintos

O mesmo símbolo `=>` aparece em dois contextos completamente diferentes em C#.

### Caso 1 — Lambda (callback inline)

```csharp
m_Colliders.RemoveAll(col => col.isTrigger);
```

Aqui, `=>` define uma função anônima passada como argumento.

### Caso 2 — Propriedade com expression body

```csharp
public List<Collider> colliders => m_Colliders;
```

Aqui, `=>` é apenas uma **sintaxe curta de propriedade**. Equivale a:

```csharp
public List<Collider> colliders
{
    get { return m_Colliders; }
}
```

> **Regra prática:** se o `=>` aparece após uma lista de parâmetros ou dentro de uma chamada de função, é lambda. Se aparece logo após o nome de uma propriedade ou método, é expression body (atalho de sintaxe).

---

## 7. Observação Final: Collision vs. Trigger no Unity

O Unity oferece duas formas distintas de detecção de contato entre objetos:

| | `OnCollision` | `OnTrigger` |
|---|---|---|
| `isTrigger` no collider | ❌ desmarcado | ✅ marcado |
| Reage a colisão física | ✅ | ❌ |
| Reage a sobreposição | ❌ | ✅ |

- Para ativar o evento de Trigger, a caixinha **`isTrigger`** do collider deve estar marcada no Inspector.
- Para configurar isso via código (sem depender do Inspector):

```csharp
GetComponent<Collider>().isTrigger = true;
```

---

## Resumo Geral

```
Callback   →  função passada como argumento para outra função
Delegate   →  definição de tipo (contrato de assinatura do callback)
Lambda     →  callback escrito inline, sem nome, com sintaxe: param => expressão

RemoveAll(teste)              →  callback nomeado
RemoveAll(a => true)          →  lambda equivalente
RemoveAll(col => col.isTrigger) →  lambda do Unity: remove colliders do tipo trigger
```

---

*Gerado a partir do desenvolvimento de `Door.cs` — Unity 6 / XR Interaction Toolkit*
