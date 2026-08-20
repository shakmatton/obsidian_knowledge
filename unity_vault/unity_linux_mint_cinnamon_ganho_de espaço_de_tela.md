
# 🧩 Otimizando espaço vertical no Unity (Linux Mint Cinnamon)

Guia completo com **duas abordagens complementares**:

1. Remover barra de título (máximo ganho)
    
2. Usar tema compacto (mais estável e elegante)
    

---

# 🎯 Objetivo

Reduzir ao máximo o desperdício de espaço vertical no Unity Editor:

- Interface mais limpa
    
- Mais área útil
    
- Melhor fluxo de trabalho
    

---

# ⚙️ Abordagem 1 — Remover barra de título (devilspie2)

## 🚀 Instalação

```bash
sudo apt install devilspie2
```

---

## 📁 Criar configuração

```bash
mkdir -p ~/.config/devilspie2
nano ~/.config/devilspie2/unity.lua
```

Conteúdo:

```lua
if (get_window_name():match("Unity") or get_application_name():match("Unity")) then
    undecorate_window()
end
```

---

## ▶️ Executar

```bash
devilspie2 &
```

---

## ✅ Resultado

- Remove a barra de título
    
- Máximo ganho de espaço
    
- Visual mais “integrado”
    

---

## 💡 Opcional: abrir maximizado

```lua
if (get_window_name():match("Unity")) then
    undecorate_window()
    maximize()
end
```

---

## 🔁 Inicialização automática

Configurações → Aplicativos de Inicialização  
Adicionar comando:

```
devilspie2
```

---

# 🔄 Remoção / Reversão

## Parar imediatamente

```bash
pkill devilspie2
```

---

## Remover da inicialização

Configurações → Aplicativos de Inicialização → remover entrada

---

## Apagar regra

```bash
rm ~/.config/devilspie2/unity.lua
```

ou:

```bash
rm -r ~/.config/devilspie2
```

---

## Desinstalar (opcional)

```bash
sudo apt remove devilspie2
```

---

## Alternativa: desativar sem apagar

```lua
-- if (get_window_name():match("Unity")) then
--     undecorate_window()
-- end
```

---

# 🎨 Abordagem 2 — Tema GTK compacto (mais estável)

## 🎯 Objetivo

- Reduzir altura da barra de título
    
- Diminuir padding de menus e botões
    
- Melhorar densidade da interface
    

---

## 🚀 Criar tema personalizado

### 1. Copiar tema base

```bash
mkdir -p ~/.themes
cp -r /usr/share/themes/Mint-Y ~/.themes/Mint-Y-Compact
```

---

## ✏️ 2. Ajustar barra de título

```bash
nano ~/.themes/Mint-Y-Compact/cinnamon/cinnamon.css
```

Procure:

```css
.titlebar {
    height: 28px;
}
```

Altere para:

```css
.titlebar {
    height: 20px;
    padding-top: 0px;
    padding-bottom: 0px;
}
```

---

## 🔘 3. Ajustar botões

Procure:

```css
.window-button {
    padding: 6px;
}
```

Substitua:

```css
.window-button {
    padding: 2px;
    min-height: 16px;
    min-width: 16px;
}
```

---

## 📦 4. Ajustar GTK (menus e headerbars)

```bash
nano ~/.themes/Mint-Y-Compact/gtk-3.0/gtk.css
```

Adicionar no final:

```css
headerbar, .titlebar {
    min-height: 20px;
    padding: 0;
}

button {
    padding-top: 2px;
    padding-bottom: 2px;
}

menubar {
    padding-top: 0px;
    padding-bottom: 0px;
}
```

---

## 🎛️ 5. Ativar tema

Configurações → Temas:

- Controles → Mint-Y-Compact
    
- Bordas de janela → Mint-Y-Compact
    

---

## ✅ Resultado

- Barra de título menor (~18–20px)
    
- Botões compactos
    
- Menus mais densos
    
- Sistema permanece estável
    

---

## ⚠️ Ajuste fino (opcional)

Para ainda mais compacto:

```css
min-height: 18px;
```

Cuidado:

- Pode dificultar cliques
    
- Pode causar glitches em alguns apps
    

---

# 🔄 Como desfazer o tema

## Voltar ao padrão

Configurações → Temas → selecione Mint-Y original

---

## Remover tema customizado

```bash
rm -r ~/.themes/Mint-Y-Compact
```

---

# 🧠 Comparação das abordagens

|Método|Espaço ganho|Estabilidade|Complexidade|
|---|---|---|---|
|Tema compacto|Médio|Alta|Média|
|Devilspie2|Máximo|Média|Baixa|

---

# 🧩 Estratégia recomendada

## ✔️ Opção 1 (equilíbrio)

- Usar apenas tema compacto
    

## ✔️ Opção 2 (máximo desempenho)

- Tema compacto + devilspie2
    

---

# 🏁 Conclusão

- Tema compacto resolve grande parte do problema sem hacks
    
- Devilspie2 entrega o máximo ganho possível
    
- Ambos são reversíveis e podem ser combinados
    

---

📌 Dica final:  
Teste primeiro o tema. Se ainda achar alto, combine com devilspie2.


===============================
=================================================

=================================================




# 🧩 Remover barra de título do Unity no Linux Mint (Cinnamon)

Guia prático para ganhar espaço vertical no Unity Editor usando `devilspie2`, com instruções completas de instalação e remoção.

---

## 🎯 Objetivo

Eliminar a barra de título da janela do Unity para:

* Ganhar mais espaço útil na tela
* Deixar a interface mais limpa
* Aproximar o comportamento de janelas “integradas”

---

## ⚙️ Como funciona

O `devilspie2` permite aplicar regras a janelas específicas (como remover bordas, maximizar, etc.) com base no nome ou aplicação.

---

# 🚀 Instalação e configuração

## 1. Instalar o devilspie2

```bash
sudo apt install devilspie2
```

---

## 2. Criar pasta de configuração

```bash
mkdir -p ~/.config/devilspie2
```

---

## 3. Criar regra para o Unity

```bash
nano ~/.config/devilspie2/unity.lua
```

Cole o conteúdo abaixo:

```lua
if (get_window_name():match("Unity") or get_application_name():match("Unity")) then
    undecorate_window()
end
```

---

## 4. Executar o devilspie2

```bash
devilspie2 &
```

---

## ✅ Resultado esperado

* O Unity abre **sem barra de título**
* Interface fica mais compacta
* Mais espaço vertical disponível

---

## 💡 (Opcional) Forçar maximização automática

Se quiser que o Unity já abra maximizado:

```lua
if (get_window_name():match("Unity")) then
    undecorate_window()
    maximize()
end
```

---

## 🔁 Inicializar automaticamente

Para evitar rodar manualmente:

1. Vá em: **Configurações → Aplicativos de Inicialização**
2. Adicione um novo item:

   * Nome: `devilspie2`
   * Comando: `devilspie2`

---

# 🔄 Como desfazer (reverter tudo)

## 1. Parar o efeito imediatamente

```bash
pkill devilspie2
```

---

## 2. Remover da inicialização

* Acesse: **Configurações → Aplicativos de Inicialização**
* Remova `devilspie2`

---

## 3. Apagar a regra

```bash
rm ~/.config/devilspie2/unity.lua
```

Ou remover tudo:

```bash
rm -r ~/.config/devilspie2
```

---

## 4. (Opcional) Desinstalar

```bash
sudo apt remove devilspie2
```

---

# 🧠 Alternativa: desativar sem apagar

Você pode apenas comentar o script:

```lua
-- if (get_window_name():match("Unity")) then
--     undecorate_window()
-- end
```

---

## 📌 Observações importantes

* Funciona melhor com o Unity maximizado
* Algumas janelas secundárias podem não seguir a regra
* Não altera permanentemente o sistema
* Pode ser ligado/desligado a qualquer momento

---

# ✔️ Resumo

| Ação           | Comando principal             |
| -------------- | ----------------------------- |
| Instalar       | `sudo apt install devilspie2` |
| Executar       | `devilspie2 &`                |
| Parar          | `pkill devilspie2`            |
| Remover config | `rm -r ~/.config/devilspie2`  |

---

Pronto — você tem um setup simples, reversível e eficaz para otimizar o espaço do Unity no Linux Mint.

