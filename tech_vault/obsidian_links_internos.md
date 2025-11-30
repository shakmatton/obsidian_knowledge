Explicação sobre como criar links entre arquivos.

* * * * *

Exemplo: O usuário possui dois arquivos abertos lado a lado no Obsidian:

-   AulaB.md
-   AulaC.md

Trechos iniciais:

-   Em AulaB: "Material Didático - Aula B: Programação em Unity com C#"
-   Em AulaC: "C# Puro vs C# Unity - Guia Completo"

Objetivo: criar um link interno entre os dois arquivos usando a palavra "Unity".

* * * * *

Como criar os links:

No arquivo AulaB.md: Substituir a palavra Unity por:  {{AulaC|Unity}}  (susbtitua chaves por colchetes)

No arquivo AulaC.md: Substituir a palavra Unity por:  {{AulaB|Unity}}  (susbtitua chaves por colchetes)

Resultado:

-   Em AulaB, ao clicar em "Unity", abre AulaC.
-   Em AulaC, ao clicar em "Unity", abre AulaB.

* * * * *

Dica extra: Se quiser manter o texto original e apenas adicionar o link: Unity ({{AulaC}}) (susbtitua chaves por colchetes)

* * * * *

Graph View no Obsidian:

Como abrir:

-   Clique no ícone de gráfico na barra lateral esquerda.
-   Ou use o atalho: Ctrl + G (Windows/Linux) ou Cmd + G (Mac).

O que aparece:

-   Cada nota é um nó (bolinha).
-   Os links criados aparecem como linhas conectando os nós.

Exploração:

-   Passe o mouse sobre um nó para destacar conexões.
-   Clique em um nó para abrir a nota.
-   Use zoom para visualizar melhor.

Configurações úteis:

-   Filtrar por tags, links ou pastas.
-   Ajustar força de atração/repulsão dos nós.
-   Usar o Graph Local para ver apenas conexões da nota atual.

* * * * *

Resultado final: As notas AulaB e AulaC estão conectadas pela palavra "Unity" dentro dos arquivos e visivelmente relacionadas no Graph View do Obsidian.

* * * * *

