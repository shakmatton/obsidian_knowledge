Em Unity, os atributos **[Serializable]** e **[SerializeField]** têm propósitos diferentes, mas ambos estão relacionados à forma como o motor lida com a serialização de dados (ou seja, salvar e expor valores no Inspector e em arquivos de cena).

### Diferença principal

| Atributo | O que faz | Quando usar |
|----------|-----------|-------------|
| **[Serializable]** | Permite que uma **classe ou struct** seja serializada pelo Unity. Isso significa que seus campos podem aparecer no Inspector e ser salvos em cenas/prefabs. | Usado em tipos personalizados (classes/structs) que não derivam de `MonoBehaviour` ou `ScriptableObject`. Exemplo: criar uma struct `WeaponStats` e querer que ela apareça no Inspector. |
| **[SerializeField]** | Força a serialização de um **campo específico** (mesmo que seja `private`). Isso faz com que o campo apareça no Inspector e seja salvo. | Usado em variáveis privadas que você quer expor no Inspector sem torná-las públicas. Ajuda a manter o encapsulamento do código. |

### Exemplos práticos

```csharp
[System.Serializable] // Permite serializar esta classe
public class WeaponStats {
    public int damage;
    public float range;
}

public class Player : MonoBehaviour {
    [SerializeField] // Exibe no Inspector mesmo sendo privado
    private int health = 100;

    public WeaponStats weapon; // Aparece no Inspector porque WeaponStats é [Serializable]
}
```

- Sem **[Serializable]**, o Unity não mostraria os campos internos de `WeaponStats` no Inspector.
- Sem **[SerializeField]**, o campo `health` privado não apareceria no Inspector, mesmo sendo salvo internamente.

### Resumindo
- **[Serializable]** → usado em **tipos** (classes/structs) para permitir que seus campos sejam serializados.  
- **[SerializeField]** → usado em **variáveis** para forçar que apareçam no Inspector, mesmo se forem privadas.  