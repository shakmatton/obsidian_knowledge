# Gravação de Tela e Otimização de Vídeo no Windows

**Data:** 03 de dezembro de 2025

---

## Ferramentas Nativas do Windows

**Usuário:**  
Me fale sobre as ferramentas do Windows para fazer tudo isso. O gravador nativo do Windows, o ScreenToGif...

**Copilot:**  
Perfeito, vamos focar nas ferramentas que você já tem à mão no Windows para gravar e transformar em GIFs 👇  

### 🎥 Gravador nativo do Windows (Xbox Game Bar)
- Atalho rápido: `Win + G` abre a Xbox Game Bar
- Função principal: gravação de tela e áudio sem precisar instalar nada
- Limitações: não exporta direto para GIF, apenas MP4

### 🖼️ ScreenToGif
- Ferramenta gratuita e leve para criar GIFs
- Editor embutido para cortar, adicionar texto, ajustar FPS
- Exporta em GIF ou MP4

### ⚖️ Comparação rápida

| Ferramenta    | Tipo de saída | Facilidade    | Melhor uso             |
|---------------|---------------|---------------|------------------------|
| Xbox Game Bar | MP4           | Muito simples | Gravar trechos da tela |
| ScreenToGif   | GIF / MP4     | Mais opções   | Criar GIFs editados    |

---

## Compatibilidade com Suítes Office Alternativas

**Usuário:**  
Consigo usar o gif gerado nas versões de powerpoint no OnlyOffice ou no LibreOffice? Quero não depender do powerpoint do windows powerpoint...

**Copilot:**  
Sim, você consegue usar GIFs animados tanto no OnlyOffice quanto no LibreOffice Impress.

- **LibreOffice Impress:** suporta GIF animado, roda automaticamente
- **OnlyOffice Presentation:** GIF aparece estático → solução é usar MP4

---

**Usuário:**  
No LibreOffice funcionou, mas no OnlyOffice o GIF fica parado.

**Copilot:**  
Isso é limitação do OnlyOffice. Ele não suporta GIF animado.
- Alternativa: converter para MP4 e inserir como vídeo
- No LibreOffice, mantenha GIFs curtos

---

**Usuário:**  
Testei MP4 no OnlyOffice e no LibreOffice. No primeiro, vídeo com faixas pretas; no segundo, não suporta MP4.

**Copilot:**  
- **OnlyOffice:** ajuste proporção antes de exportar (16:9) para evitar faixas pretas
- **LibreOffice:** não suporta MP4, mas aceita GIF ou formatos como OGV/WebM

---

## Comparação: GIF vs MP4

**Usuário:**  
Meu GIF de 11s ficou com 6 MB, mas o MP4 de 21s ficou com 1 MB.

**Copilot:**  
Isso acontece porque GIF não tem compressão temporal, enquanto MP4 usa H.264/H.265.
- GIF → pesado, limitado a 256 cores
- MP4 → leve, alta qualidade
- Estratégia: GIFs curtos, MP4s para trechos longos

---

**Usuário:**  
Vou adotar MP4.

**Copilot:**  
Boa escolha!
- Vantagens: leve, compatível, qualidade superior
- Configuração ideal: 16:9, 720p, 15–20 fps, bitrate 1000–1500 kbps

---

## Configuração de Proporção 16:9

**Usuário:**  
Consigo ajustar 16:9 no ScreenToGif exportando em MP4?

**Copilot:**  
Sim.
- Grave já em 16:9
- Use cortar/redimensionar no editor
- Exporte em MP4 (H.264)

---

## Usando OBS Studio

**Usuário:**  
E se eu usar o OBS Studio?

**Copilot:**  
Ótima alternativa.
- Controle total de resolução e proporção
- Exporta em MP4 leve
- Configuração: 720p, 15–20 fps, bitrate 1000–2000 kbps

---

**Usuário:**  
Após interromper gravação no OBS, já salva automático?

**Copilot:**  
Sim.
- Clique em "Parar gravação" → arquivo salvo automaticamente
- Pasta padrão: Vídeos
- Formato: MKV (padrão) ou MP4 (configurável)

---

## Otimização de Tamanho de Arquivo

**Usuário:**  
Meu vídeo de 10s ficou com 10 MB. Como otimizar?

**Copilot:**  
- Reduza FPS para 15–20
- Bitrate: 1000–1500 kbps
- Saída: 720p
- Encoder: NVENC H.264

---

**Usuário:**  
Resolução base 1920×1080 e saída 1280×720. Está bom?

**Copilot:**  
Sim.
- Base 1080p, saída 720p → equilíbrio perfeito
- Mantém qualidade, reduz tamanho

---

**Usuário:**  
Saída em 1500 kbps, NVENC H.264.

**Copilot:**  
Excelente.
- NVENC usa GPU → rápido e leve
- 1500 kbps em 720p → ótimo equilíbrio
- Se quiser mais leve, reduza para 1000–1200 kbps

---

## Configuração de Áudio no OBS

**Usuário:**  
Estou ouvindo música no headphone, mas não quero que apareça na gravação.

**Copilot:**  
Mute o **Desktop Audio** no OBS.
- Assim, música não entra
- Se quiser som do programa, configure saídas separadas no Windows

---

## Resultados de Otimização

**Usuário:**  
Novo teste reduziu para ~2,3 MB. Dá pra otimizar mais?

**Copilot:**  
Sim.
- FPS: 15
- Bitrate: 1000–1200 kbps
- Resolução: 720p
- Isso deve cair para ~1 MB em 10s

---

## Comparação: ScreenToGif vs OBS Studio

**Usuário:**  
ScreenToGif gerou MP4 mais leve e com qualidade superior.

**Copilot:**  
Diferença é configuração.
- ScreenToGif já vem com presets otimizados
- OBS precisa ajuste manual (FPS + bitrate)
- Com ajustes, OBS pode gerar arquivos tão leves quanto ScreenToGif

---

## Resumo das Melhores Práticas

### Configurações Recomendadas para MP4
- **Proporção:** 16:9
- **Resolução:** 720p (1280×720)
- **FPS:** 15–20
- **Bitrate:** 1000–1500 kbps
- **Encoder:** NVENC H.264 (quando disponível)

### Escolha de Ferramenta
- **ScreenToGif:** Simplicidade e presets otimizados
- **OBS Studio:** Controle avançado e personalizações
- **Xbox Game Bar:** Gravação rápida sem configuração

### Compatibilidade com Apresentações
- **LibreOffice Impress:** Use GIF (suporta animação)
- **OnlyOffice Presentation:** Use MP4 em 16:9 (evita faixas pretas)

### Estratégia de Formato
- **GIF:** Apenas para animações muito curtas (< 5s)
- **MP4:** Para qualquer conteúdo com mais de 5s
- **Razão:** MP4 é muito mais leve e com melhor qualidade