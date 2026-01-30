# Convenções de Nomenclatura em C# e Unity

## Pergunta 1: O que significa o prefixo `m_` nas variáveis?

**Contexto:** No código do `GravityProvider` da Unity, aparecem variáveis como:

- `m_UseGravity`
- `m_TerminalVelocity`
- `m_SphereCastRadius`

### Resposta

O `m_` que aparece antes dos nomes das variáveis não é algo imposto pela linguagem C# ou pelo Unity em si — é apenas uma **convenção de nomenclatura** usada pelos desenvolvedores da Unity (e por muitos outros times) para indicar que aquela variável é um **campo privado da classe** (um _member variable_).

#### O que significa na prática:

- `m_` **= "member"** → indica que a variável pertence à instância da classe (um campo), e não é uma variável local ou estática.
    
- É uma forma de diferenciar rapidamente:
    
    - **Campos privados** → `m_UseGravity`
    - **Propriedades públicas** → `useGravity`
    - **Variáveis locais** → geralmente sem prefixo, ex: `velocity` dentro de um método.

#### Exemplo simplificado:

```csharp
public class Player
{
    // Campo privado (com prefixo m_)
    private int m_Health = 100;

    // Propriedade pública que expõe o campo
    public int health
    {
        get => m_Health;
        set => m_Health = value;
    }

    public void TakeDamage(int amount)
    {
        // Variável local (sem prefixo)
        int newHealth = m_Health - amount;
        m_Health = newHealth;
    }
}
```

#### Por que usar esse padrão?

- **Clareza**: ao bater o olho em `m_Health`, já se sabe que é um campo da classe.
- **Consistência**: facilita distinguir entre variáveis locais, parâmetros e propriedades.
- **Estilo da Unity**: muitos scripts internos da Unity seguem essa convenção, então é comum vê-la em exemplos oficiais.

> **Em resumo:** o `m_` não tem efeito técnico, é apenas um **prefixo de estilo** para indicar que a variável é um **campo privado da classe**.

---

## Pergunta 2: Outras convenções e por que não usar `health.get()`?

### Convenções comuns no código Unity

Além do `m_` que você já viu, há outros prefixos usados nos scripts oficiais:

|Prefixo|Significado|Exemplo|
|---|---|---|
|`m_`|**Member field** (campo privado da instância)|`m_UseGravity`|
|`s_`|**Static field** (compartilhado por todas as instâncias)|`s_Instance`|
|`k_`|**Constant** ou valor fixo usado como configuração|`k_DefaultGravity`|
|`U`|Às vezes usado para **UnityEvent** ou variáveis ligadas a Unity|`UOnJump`|

### Diferença entre acessar diretamente o campo (`m_Health`) e usar a propriedade (`health`)

#### Primeiro exemplo (acesso direto ao campo):

```csharp
public void TakeDamage(int amount)
{
    // Variável local (sem prefixo)
    int newHealth = m_Health - amount;
    m_Health = newHealth;
}
```

- Aqui você acessa diretamente o **campo privado** `m_Health`.
- É simples e rápido, mas ignora qualquer lógica extra que poderia existir na **propriedade pública** `health`.

#### Segundo exemplo (via propriedade):

**Nota:** Em C#, não se usa `get()` explicitamente. O acesso à propriedade é feito assim:

```csharp
public void TakeDamage(int amount)
{
    int newHealth = health - amount;
    health = newHealth;
}
```

- Isso chamaria automaticamente o **getter** e depois o **setter** da propriedade `health`.
- Se o `set` ou `get` tiver lógica adicional (validação, eventos, etc.), essa lógica seria executada.

#### Por que o código original usa `m_Health`?

- Muitas vezes, dentro da própria classe, os devs preferem acessar o **campo privado** diretamente, porque sabem que não precisam da lógica extra da propriedade.
- Usar a propriedade pode ser redundante ou até causar efeitos colaterais indesejados (por exemplo, disparar eventos toda vez que o valor muda).
- Já para código **externo à classe**, o acesso deve ser via propriedade (`health`), porque o campo privado não é visível fora.

### Resumo

- `m_` = _member privado_
- Dentro da classe, usa-se o campo direto (`m_Health`)
- Fora da classe, usa-se a propriedade (`health`)
- Em C#, propriedades são acessadas diretamente sem `.get()` ou `.set()`