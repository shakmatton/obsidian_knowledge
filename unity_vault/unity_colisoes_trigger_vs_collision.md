# Unity — Colisão entre Nave e Asteroide: Lições Aprendidas

## Contexto

Minigame com perspectiva top-down em cena XR, simulando um jogo de nave 2D. O objetivo era destruir asteroides ao colidir com a nave, usando um script genérico (`PrefabDestroyer`) capaz de ser reutilizado em outros contextos (bala vs inimigo, etc).

---

## O script original

```csharp
public class PrefabDestroyer : MonoBehaviour
{
    [SerializeField] private GameObject prefab;

    private void OnCollisionEnter(Collision collision)
    {
        if (collision.gameObject.CompareTag(prefab.tag))
        {
            Destroy(collision.gameObject);
        }
    }
}
```

O asteroide atravessava a nave em vez de ser destruído — `OnCollisionEnter` nunca disparava.

---

## Diagnóstico: por que o OnCollisionEnter não disparava?

### 1. Rigidbody ausente
`OnCollisionEnter` exige que pelo menos um dos objetos tenha um `Rigidbody`. Sem ele, a física não processa a colisão.

### 2. Collision Detection Mode inadequado
O modo padrão (`Discrete`) verifica colisões por "snapshots" a cada frame. Objetos rápidos podem pular de um lado para o outro da nave entre frames sem nunca se sobrepor. Solução: usar `Continuous` ou `Continuous Dynamic` no Rigidbody do asteroide.

### 3. Is Trigger marcado
Se qualquer um dos dois colliders tiver `Is Trigger = true`, o Unity não processa colisão física — ele chama `OnTriggerEnter` em vez de `OnCollisionEnter`.

### 4. Layer Collision Matrix
Em *Edit → Project Settings → Physics → Layer Collision Matrix*, as layers dos dois objetos precisam estar habilitadas para colidir. Layers customizadas **não são obrigatórias** — objetos na layer `Default` já colidem entre si.

### 5. O problema real: dois Rigidbodies Kinematic

| Nave | Asteroide | `OnCollisionEnter` dispara? |
|---|---|---|
| Rigidbody normal | Rigidbody normal | ✅ Sim |
| Rigidbody Kinematic | Rigidbody normal | ✅ Sim |
| Rigidbody normal | Rigidbody Kinematic | ✅ Sim |
| Rigidbody Kinematic | Rigidbody Kinematic | ❌ **Não** |

A nave era kinematic (controlada por `transform.position`) e o asteroide também era kinematic. Dois objetos kinematic **não geram eventos de colisão física entre si**.

---

## OnCollisionEnter vs OnTriggerEnter

| | `OnCollisionEnter` | `OnTriggerEnter` |
|---|---|---|
| O que faz | Física real — objetos se empurram | Detecção de sobreposição — objetos se atravessam |
| Requisito | Nenhum collider pode ser Trigger | Pelo menos um collider com `Is Trigger = true` |
| Quando usar | Objetos que devem quicar/empurrar | Áreas de efeito, killzones, destruição sem física |

### Exemplos práticos

| Situação | Método recomendado |
|---|---|
| Objetos que se empurram fisicamente | `OnCollisionEnter` |
| Bala destruindo inimigo (sem ricochete) | `OnTriggerEnter` |
| Nave colidindo com asteroide (destruição) | `OnTriggerEnter` |
| Checkpoint, killzone, área de efeito | `OnTriggerEnter` |

---

## Solução adotada — Opção B (Trigger)

Marcar `Is Trigger = true` no **Box Collider da nave** e usar `OnTriggerEnter` no script:

```csharp
public class PrefabDestroyer : MonoBehaviour
{
    [SerializeField] private GameObject prefab;

    private void OnTriggerEnter(Collider other)
    {
        if (prefab == null) return;

        if (other.CompareTag(prefab.tag))
        {
            Destroy(other.gameObject);
        }
    }
}
```

Nenhuma alteração necessária em `Mover.cs` ou `NewSpaceshipController.cs`.

---

## Por que não mexer na nave

`NewSpaceshipController` move a nave via `transform.position` com `Mathf.Clamp` para limitar bordas. Remover o kinematic da nave quebraria esse comportamento, pois o motor de física passaria a aplicar gravidade e forças externas sobre ela.

---

## Solução alternativa — Opção A (Collision, didática)

Para fazer `OnCollisionEnter` funcionar com os scripts existentes:

1. Desativar `Is Kinematic` nos asteroides (manter `Use Gravity = false`)
2. Reescrever `Mover.cs` usando `rb.MovePosition` no `FixedUpdate`:

```csharp
private Rigidbody rb;

private void Start()
{
    rb = GetComponent<Rigidbody>();
}

private void FixedUpdate()
{
    rb.MovePosition(rb.position + transform.forward * speed * acceleration * Time.fixedDeltaTime);
}
```

> Mover um Rigidbody não-kinematic via `transform.position` no `Update` "teleporta" o objeto a cada frame, ignorando o sistema de física — por isso o asteroide atravessava outros objetos.

---

## Boa prática: null check no prefab

```csharp
if (prefab == null) return;
```

Sem essa guarda, se o campo `Prefab` estiver vazio no Inspector, `prefab.tag` lança uma `NullReferenceException` silenciosa e o `Destroy` nunca acontece.

---

## Próximas versões planejadas

- **Versão 1** ✅ Nave toca no asteroide e destrói ele
- **Versão 2** Nave atira no asteroide e destrói ele
- **Versão 3** Nave atira no asteroide; asteroide toca em asteroide e ambos são destruídos
