# Estudo sobre Unity XR Interaction Toolkit e Singleton

## 📌 Introdução
Este documento reúne explicações sobre o **XR Interaction Toolkit**, o papel do **XR Interaction Manager**, e o uso do padrão **Singleton** em Unity.
Foi elaborado para consulta e estudo offline.

---

## 🌐 XR Interaction Toolkit
- Instalado via **Unity Package Manager**.
- Inclui **sample assets** como mapas de ação e bindings de controle.
- Configuração típica:
  - **XR Origin** → define a posição e referência do usuário no espaço.
  - **Interactors** → quem interage (ex.: XR Ray Interactor, XR Direct Interactor).
  - **Interactables** → objetos que podem ser manipulados (ex.: XR Grab Interactable, XR Simple Interactable).

---

## 🧩 XR Interaction Manager
### O que é
- **Cérebro central** que conecta Interactors e Interactables.
- Coordena eventos de interação (select, hover, grab).
- **Registro automático**: Interactors e Interactables se registram nele.
- **Obrigatório**: toda cena precisa de pelo menos um Manager.

### Fluxo de funcionamento
1. **Interactor** envia evento (ex.: apontar ou pegar).
2. **Interaction Manager** recebe e decide qual Interactable responde.
3. **Interactable** executa a ação (pegar, soltar, ativar).

### Exemplos práticos
- **XR Direct Interactor** pega um cubo com **XR Grab Interactable**.
- **XR Ray Interactor** ativa um botão com **XR Simple Interactable**.

---

## ⚖️ Singleton e XR Interaction Manager
### O que é Singleton
- Padrão que garante **uma única instância** de uma classe.
- Permite acesso global sem criar novos objetos.

### Por que usar Singleton no Manager
- **Facilidade de acesso**: qualquer script pode chamar o Manager.
- **Consistência**: todos usam o mesmo ponto de controle.
- **Clareza para equipe**: explícito que há apenas um Manager.

### Exemplo de código
```csharp
public class XRInteractionManagerSingleton : MonoBehaviour
{
    public static XRInteractionManagerSingleton Instance { get; private set; }

    private XRInteractionManager manager;

    void Awake()
    {
        if (Instance != null && Instance != this)
        {
            Destroy(gameObject); // Evita duplicatas
            return;
        }
        Instance = this;
        manager = GetComponent<XRInteractionManager>();
    }

    public XRInteractionManager GetManager()
    {
        return manager;
    }
}
```

**Uso:**
```csharp
XRInteractionManagerSingleton.Instance.GetManager();
```

---

## 🏢 Analogia empresarial

- **Singleton** → um único CEO centraliza todas as decisões.
- **Múltiplos Managers** → vários gerentes de departamentos cuidam de áreas diferentes.

---

## ⚠️ Cuidados

- Singleton é ótimo para **projetos pequenos/médios** → clareza e simplicidade.
- Em **projetos grandes/complexos**, múltiplos Managers podem ser necessários → modularidade e escalabilidade.
- Documente bem a arquitetura escolhida para evitar confusão na equipe.

---

## 🔎 Detalhes sobre `Instance` e métodos

- `Instance` é uma **propriedade estática** que guarda a referência do objeto Singleton.
- O método `GetManager()` é **de instância**, não estático.
- O operador `.` encadeia:
  - Primeiro acessa o objeto em `Instance`.
  - Depois chama o método desse objeto.

### Analogia

- Classe = fábrica.
- `Instance` = caixa única que guarda o produto.
- `GetManager()` = funcionalidade do produto dentro da caixa.

---

## ✅ Resumo final

- O **XR Interaction Manager** é o núcleo das interações XR.
- O **Singleton** garante acesso único e global, mas pode limitar modularidade.
- **Decisão prática**:
  - Use Singleton em projetos simples.
  - Permita múltiplos Managers em projetos complexos.
- O mais importante é **clareza para a equipe** e **documentação da arquitetura**.