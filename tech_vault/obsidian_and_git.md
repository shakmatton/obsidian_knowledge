# 📘 Resumo da Conversa - Obsidian & Git

## 1. Obsidian Git Plugin
- Instalação via **Community Plugins**.
- Configurações principais:
  - Auto pull (puxar alterações ao abrir).
  - Auto push (enviar alterações automaticamente).
  - Intervalo de sincronização.
  - Mensagens de commit automáticas.
- Útil para sincronizar notas entre Linux, Windows 10 e Windows 11 sem usar terminal.

---

## 2. Inserindo arquivos no Vault
- O *vault* é apenas uma pasta comum.
- Para adicionar `aula.md`, basta mover/copiar o arquivo para dentro da pasta `tech_vault`.
- O Obsidian reconhece automaticamente e mostra na barra lateral.

---

## 3. Abrir arquivos `.md` com Obsidian
- Localizar o executável `Obsidian.exe` (Windows).
- Associar `.md` ao Obsidian via **Abrir com → Escolher outro aplicativo → Procurar Obsidian.exe**.
- Não há problema se o Obsidian estiver em pasta diferente, basta apontar corretamente.

---

## 4. Criar e editar notas
- Editar diretamente no Obsidian.
- Sintaxe Markdown:
  - `# Título`
  - `## Subtítulo`
  - `- Listas`
  - `**Negrito**`, `*itálico*`, `[links](url)`
  - Blocos de código com ```.

---

## 5. Canvas no Obsidian
- Canvas permite criar diagramas e fluxogramas com notas.
- Atenção: editar dentro do Canvas altera o arquivo original.
- Soluções:
  - Usar **links internos**: `[[aulaB#Introdução]]`.
  - Usar **embeds**: `![[aulaB#Introdução]]` → mostra trecho renderizado dentro da caixa.
  - Criar subpastas com arquivos separados (Introdução, Revisão, etc.).
  - Usar Git para versionamento e evitar perda de conteúdo.

---

## 6. Salvamento automático
- Obsidian salva tudo automaticamente.
- Arquivos `.md` e `.canvas` são atualizados em tempo real na pasta do *vault*.
- Não existe botão de salvar.

---

## 7. Canvas e compartilhamento
- Canvas é salvo como arquivo `.canvas` (JSON).
- Compartilhar via Git: incluir o `.canvas` no commit.
- Exportar como imagem (PNG) → converter para PDF se necessário.


## 8. Criar um arquivo vazio via terminal no Windows 10
- Abra o **Prompt de Comando** (cmd) ou **PowerShell**.
- No **cmd**, use:
  ```cmd
  type nul > nome_do_arquivo.md


---

## 9. GitHub e repositório global
- Inicializar Git na pasta **Obsidian Repository**.
- Criar repositório remoto no GitHub (ex.: `obsidian_knowledge`).
- Conectar:
  ```bash
  git remote add origin https://github.com/SEU_USUARIO/obsidian_knowledge.git
  git branch -M main
  git push -u origin main
