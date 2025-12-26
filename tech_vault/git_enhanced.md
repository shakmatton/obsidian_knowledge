# Git Enhanced - Guia Completo para Projetos Unity

## 📍 Localização dos Projetos Unity no Windows 11

Por padrão:
```
C:\Users\[SeuNome]\Documents\Unity Projects\
```

Cada projeto tem subpastas: `Assets`, `ProjectSettings`, `Packages`, etc.

---

## 🚀 Configuração Inicial do Git para Unity

### 1. Criar Arquivo .gitignore

Na raiz do projeto Unity, crie `.gitignore`:

```gitignore
# Unity generated
[Ll]ibrary/
[Tt]emp/
[Oo]bj/
[Bb]uild/
[Bb]uilds/
[Ll]ogs/
[Uu]ser[Ss]ettings/

# MemoryCaptures
[Mm]emoryCaptures/

# Asset meta data
*.pidb
*.booproj
*.svd
*.pdb
*.mdb
*.opendb
*.VC.db

# Unity3D generated meta files
*.pidb.meta
*.pdb.meta
*.mdb.meta

# Unity3D generated file on crash reports
sysinfo.txt

# Builds
*.apk
*.aab
*.unitypackage
*.app

# Crashlytics generated file
crashlytics-build.properties

# Packed Addressables
[Aa]ssets/[Aa]ddressable[Aa]ssets[Dd]ata/*/*.bin*

# Temporary auto-generated Android Assets
[Aa]ssets/[Ss]treamingAssets/aa.meta
[Aa]ssets/[Ss]treamingAssets/aa/*

# JetBrains Rider
.idea/
*.sln.iml
*.sln.DotSettings.user
```

### 2. Configurar Unity para Git

**Unity 6.x:**
- `Edit → Project Settings → Version Control`
- Mode: **Visible Meta Files**

**Ou editar manualmente** `ProjectSettings/EditorSettings.asset`:
```yaml
m_ExternalVersionControlSupport: Visible Meta Files
m_SerializationMode: 2
```

---

## 🔧 Configuração de Identidade

### Configuração Local (Recomendada para Separar Trabalho/Casa)

**Na pasta do projeto:**
```bash
# Configurar APENAS para este projeto (sem --global)
git config user.name "Seu Nome"
git config user.email "seu-username@users.noreply.github.com"
```

### Configuração Global (Aplica a Todos os Projetos)

```bash
git config --global user.name "Seu Nome"
git config --global user.email "seuemail@exemplo.com"
```

### Verificar Configurações

```bash
# Ver configuração local do projeto
git config user.name
git config user.email

# Ver configuração global
git config --global user.name
git config --global user.email
```

---

## 🌐 Criar Repositório no GitHub

1. Acesse https://github.com
2. Clique em **"+"** → **"New repository"**
3. Nomeie o repositório
4. Escolha público ou privado
5. **NÃO** marque "Add a README file"
6. Clique em **"Create repository"**

---

## 💻 Comandos Git Essenciais

### Inicializar Repositório Local

```bash
# Navegar até o projeto
cd "C:\Users\[SeuNome]\Documents\Unity Projects\[NomeDoProjeto]"

# Inicializar Git
git init

# Adicionar todos os arquivos
git add .

# Primeiro commit
git commit -m "Initial commit"

# Conectar ao GitHub
git remote add origin https://github.com/seu-usuario/seu-repositorio.git

# Renomear branch para main (se necessário)
git branch -M main

# Enviar para GitHub
git push -u origin main
```

### Comandos do Dia a Dia

```bash
# Ver status das mudanças
git status

# Adicionar mudanças específicas
git add Assets/Scripts/MeuScript.cs

# Adicionar todas as mudanças
git add .

# Fazer commit
git commit -m "Descrição das mudanças"

# Enviar para GitHub
git push

# Baixar mudanças do GitHub
git pull

# Ver histórico de commits
git log --oneline

# Ver diferenças não commitadas
git diff
```

### Obter Link do Repositório

```bash
# Ver URL do repositório remoto
git remote get-url origin
```

---

## 🔀 Trabalhando Entre Múltiplas Máquinas

### Clonar Projeto em Nova Máquina

```bash
# Navegar até a pasta de projetos
cd ~/Documents/Unity Projects/

# Clonar repositório
git clone https://github.com/seu-username/nome-repositorio.git

# Configurar identidade local (se necessário)
cd nome-repositorio
git config user.name "Seu Nome"
git config user.email "seuemail@exemplo.com"
```

### Workflow: Trabalho ↔ Casa

**Antes de sair do trabalho:**
```bash
git add .
git commit -m "Progresso do dia"
git push
```

**Ao chegar em casa:**
```bash
git pull
# Continue trabalhando...
git add .
git commit -m "Trabalho em casa"
git push
```

**De volta ao trabalho:**
```bash
git pull
# Continue trabalhando...
```

---

## 🪟🐧 Normalização de Linha (Windows ↔ Linux)

### No Windows
```bash
git config --global core.autocrlf true
```

### No Linux
```bash
git config --global core.autocrlf input
```

**Resultado:** O Git normaliza automaticamente os finais de linha (CRLF no Windows, LF no Linux). Não causa problemas no Unity!

---

## ⚠️ Problemas Comuns e Soluções

### Warning: LF will be replaced by CRLF

**É normal!** O Git está normalizando finais de linha. Configure:
```bash
git config core.autocrlf true
```

### Author identity unknown

Configure seu nome e email:
```bash
git config user.name "Seu Nome"
git config user.email "seuemail@exemplo.com"
```

### Pasta .idea do Rider foi enviada

Adicione ao `.gitignore`:
```gitignore
.idea/
*.sln.iml
*.sln.DotSettings.user
```

Remova do Git (mantém no disco):
```bash
git rm -r --cached .idea
git commit -m "Remove Rider .idea folder"
git push
```

### Conflitos ao fazer Pull

```bash
# Ver arquivos em conflito
git status

# Opção 1: Manter suas mudanças
git checkout --ours caminho/arquivo

# Opção 2: Aceitar mudanças remotas
git checkout --theirs caminho/arquivo

# Após resolver conflitos
git add .
git commit -m "Resolve merge conflicts"
git push
```

---

## 🔐 Privacidade e Separação Trabalho/Pessoal

### Usar Email Noreply do GitHub

1. GitHub → Settings → Emails
2. Copie: `seu-username@users.noreply.github.com`
3. Use na configuração:

```bash
git config user.email "seu-username@users.noreply.github.com"
```

### Estratégia de Múltiplas Identidades

**Máquina do Trabalho:**
```bash
cd projeto-trabalho
git config user.email "seu-username@users.noreply.github.com"
```

**Máquina de Casa:**
```bash
cd projeto-trabalho
git config user.email "seuemail@pessoal.com"
```

Ambos os emails vinculados à mesma conta GitHub!

---

## 📊 Branches (Ramificações)

### Comandos Básicos

```bash
# Criar nova branch
git branch nome-feature

# Mudar para branch
git checkout nome-feature

# Criar e mudar para branch (atalho)
git checkout -b nome-feature

# Ver todas as branches
git branch -a

# Voltar para main
git checkout main

# Mesclar branch na main
git merge nome-feature

# Deletar branch local
git branch -d nome-feature
```

---

## 🔄 Desfazer Mudanças

```bash
# Desfazer mudanças não commitadas em arquivo
git checkout -- caminho/arquivo

# Desfazer último commit (mantém mudanças)
git reset --soft HEAD~1

# Desfazer último commit (descarta mudanças)
git reset --hard HEAD~1

# Reverter commit específico (cria novo commit)
git revert <hash-do-commit>
```

---

## 📦 Repositórios Privados vs Públicos

### Tornar Repositório Privado

1. GitHub → Seu repositório → **Settings**
2. Role até **Danger Zone**
3. **Change visibility** → **Make private**

**Privado:** Só você e colaboradores convidados podem ver  
**Público:** Qualquer pessoa pode ver e clonar

---

## 📝 Boas Práticas

### Mensagens de Commit

✅ **Boas:**
- `"Adiciona sistema de movimento do jogador"`
- `"Corrige bug no sistema de colisão"`
- `"Refatora código do PlayerController"`

❌ **Ruins:**
- `"fix"`
- `"teste"`
- `"asdfasdf"`

### Frequência de Commits

- Commit pequeno e frequente > commit grande e raro
- Faça commit sempre que completar uma funcionalidade
- Faça push no fim do dia de trabalho

### O Que NÃO Commitar

- Pastas `Library/`, `Temp/`, `Obj/` (já no `.gitignore`)
- Arquivos de build (`.exe`, `.apk`)
- Configurações pessoais do editor
- Arquivos temporários

---

## 🔐 Autenticação com GitHub

### Opção 1: HTTPS com Personal Access Token (Mais Simples)

#### Criar Personal Access Token

1. GitHub → foto de perfil → **Settings**
2. **Developer settings** → **Personal access tokens** → **Tokens (classic)**
3. **Generate new token (classic)**
4. Nome: "Token Máquina Trabalho"
5. Expiration: escolha o período (90 days, 1 year, etc.)
6. Marque: ✅ `repo` (acesso completo a repositórios)
7. **Generate token**
8. **COPIE O TOKEN AGORA!** (não aparecerá novamente)

#### Usar Token

```bash
# Configurar repositório para HTTPS
git remote set-url origin https://github.com/seu-username/seu-repositorio.git

# Fazer push
git push -u origin main

# Quando pedir credenciais:
# Username: seu-username-github
# Password: cole-o-token (não sua senha!)
```

#### Salvar Credenciais

```bash
# Para não digitar toda vez
git config --global credential.helper store
```

### Opção 2: SSH (Mais Prático Depois de Configurar)

#### 1. Verificar Se Já Tem Chave SSH

```bash
ls -al ~/.ssh
```

Se aparecer `id_ed25519.pub` ou `id_rsa.pub`, pule para o passo 3.

#### 2. Gerar Chave SSH

```bash
# Gerar chave
ssh-keygen -t ed25519 -C "seu-email@exemplo.com"

# Perguntas:
# - "Enter file...": aperte Enter (usa padrão)
# - "Enter passphrase": pode deixar vazio ou criar senha extra
```

#### 3. Copiar Chave Pública

```bash
# Exibir chave pública
cat ~/.ssh/id_ed25519.pub

# Copie TUDO que aparecer (de ssh-ed25519 até o final)
```

#### 4. Adicionar no GitHub

1. GitHub → foto de perfil → **Settings**
2. **SSH and GPG keys**
3. **New SSH key**
4. **Title:** "Máquina Trabalho"
5. **Key:** cole a chave pública
6. **Add SSH key**

#### 5. Testar Conexão

```bash
ssh -T git@github.com

# Deve aparecer:
# Hi seu-username! You've successfully authenticated...
```

#### 6. Configurar Repositório para SSH

```bash
# Trocar URL para SSH (use SEU USERNAME REAL do GitHub!)
git remote set-url origin git@github.com:seu-username-real/nome-repositorio.git

# Verificar
git remote -v

# Push sem precisar digitar senha/token!
git push -u origin main
```

#### Troubleshooting SSH

Se der "Permission denied":
```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
```

### ⚠️ IMPORTANTE: Username Real vs Nome de Commit

- **Para autenticação (login):** Use sempre seu **username REAL do GitHub**
- **Para commits (identificação):** Pode usar qualquer nome (`pti_user`, `Dev`, etc.)

```bash
# Configuração de commit (pode ser pseudônimo)
git config user.name "pti_user"
git config user.email "pti_user@users.noreply.github.com"

# Mas para autenticar/push, use username REAL:
git remote set-url origin git@github.com:SEU-USERNAME-REAL/repo.git
```

---

## 📁 Arquivos .csproj e .sln - Manter ou Ignorar?

### Recomendação: **MANTER no Repositório**

**Por quê?**
- ✅ Unity regenera automaticamente, mas leva tempo
- ✅ IDEs (Rider/Visual Studio) precisam deles
- ✅ Economiza tempo, especialmente em máquinas mais lentas
- ✅ Facilita trabalho em múltiplas máquinas

### O Que Ignorar

Adicione ao `.gitignore` **apenas** configurações pessoais:

```gitignore
# User-specific files (Rider/Visual Studio)
*.suo
*.user
*.userosscache
*.sln.docstates
*.sln.DotSettings.user
.vs/
```

### Resultado

- ✅ `.sln` e `.csproj` → **no Git** (economiza regeneração)
- ✅ Configurações pessoais → **ignoradas** (evita conflitos)

---

## 🔗 Links Úteis

- [Git Documentation](https://git-scm.com/doc)
- [GitHub Docs](https://docs.github.com)
- [GitHub SSH Documentation](https://docs.github.com/en/authentication/connecting-to-github-with-ssh)
- [Unity Version Control](https://docs.unity3d.com/Manual/ExternalVersionControlSystemSupport.html)
- [Anthropic Claude Prompting](https://docs.claude.com/en/docs/build-with-claude/prompt-engineering/overview)

---

## 🎯 Checklist Rápido: Novo Projeto Unity

- [ ] Criar `.gitignore` na raiz do projeto (Unity + Rider)
- [ ] Configurar Unity: Version Control → Visible Meta Files
- [ ] `git init`
- [ ] `git config user.name` e `user.email` (local ou global)
- [ ] `git add .`
- [ ] `git commit -m "Initial commit"`
- [ ] Criar repositório no GitHub
- [ ] Configurar autenticação (HTTPS com token OU SSH)
- [ ] `git remote add origin <url>`
- [ ] `git push -u origin main`
- [ ] Verificar se `.idea/` foi ignorado (Rider)
- [ ] Confirmar que `.sln` e `.csproj` estão no repositório

---

**Última atualização:** 26/12/2025  
**Versão Unity:** 6.2  
**Sistema:** Windows 11 / Linux  
**IDEs testadas:** Rider, Visual Studio