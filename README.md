# truco-score

Placar de truco paulista com narrador. Uma página só (`index.html`), sem build
e sem dependência: abre no celular, guarda tudo no `localStorage` e funciona
offline.

## Rodar

Os clipes do narrador são carregados de `audios/`. Abrir o `index.html` direto
do disco funciona (há um mapa de clipes embutido como fallback), mas o jeito
recomendado é servir a pasta, porque aí o `audios/manifest.json` é lido de
verdade e o navegador cacheia os áudios:

    python3 -m http.server 8000
    # http://localhost:8000

## Narrador

Quatro estilos, trocáveis a qualquer momento na barra abaixo do placar —
inclusive no meio da partida:

| chip      | pasta       | voz                                    |
|-----------|-------------|----------------------------------------|
| Sério     | `serio/`    | Lucas — narrador grave                 |
| Cômico    | `comico/`   | Mário — animado, deboche leve          |
| Bar       | `bar_leve/` | Otto — gritado, sem palavrão           |
| Bar 18+   | `bar/`      | Otto — gritado, **linguagem adulta**   |

O Bar 18+ pede confirmação na primeira vez. O 🔊 à esquerda liga e desliga o
narrador sem perder o estilo escolhido. O botão à direita troca a velocidade da
voz (1× → 1,25× → 1,5× → 2×, padrão 1,25×); o tom é preservado, e a troca vale
na hora, inclusive para o clipe que está tocando. Volume, "torce para" (define
de quem é a `vitoria` e de quem é a `derrota`) e "falar placar" ficam em ⚙ →
Narrador.

### Quando cada fala dispara

| evento            | gatilho na tela                                         |
|-------------------|---------------------------------------------------------|
| `inicio`          | "Nova partida"                                          |
| `truco`/`seis`/`nove`/`doze` | chip de valor da mão                         |
| `mao_vencida`     | "Ganhou a mão"                                          |
| `correu`          | "Correu · <time>" — quem correu é o time apontado       |
| `cangou`          | "Cangou"                                                |
| `mao_de_onze`     | um time chega a 11 (uma vez por partida, por time)      |
| `mao_escura`      | os dois em 11                                           |
| `placar_apertado` | diferença ≤ 1 com alguém em 6+, de vez em quando        |
| `vitoria`/`derrota` | alguém fecha em 12                                    |

Cada evento tem 3 variações e o app sorteia sem repetir a última — um "Truco!"
idêntico toda mão faz o usuário desligar o som na primeira meia hora.

Cada toque dispara uma fala, e **toque novo corta a fala anterior inteira**:
pediu truco e logo depois seis, o "Seis!" sai na hora. O resultado de uma mão é
uma fala em sequência — clipe do evento, comentário da situação (se houver) e
o placar —, e é essa sequência toda que o próximo toque interrompe.

### Placar falado

Depois de cada mão o narrador fala o placar ("Nós, 5. Eles, 3." ou "Empate,
5 a 5."; no fim, "Placar final. …"). Desfazer e −1 também falam o placar
corrigido. Como os nomes dos times são livres, essa parte usa a voz do próprio
aparelho (Web Speech API, `pt-BR`), não um clipe gravado — soa diferente do
narrador e depende de o aparelho ter voz em português. Desliga em ⚙ → Falar
placar.

## Aparência

A cor do painel (⚙ → Aparência) define a paleta inteira: cards, bordas e
textos saem do mesmo matiz. Fundo claro vira tema claro com letra escura, fundo
escuro mantém letra clara, e toda cor de texto — inclusive a dos times — é
ajustada até ter contraste de leitura com o fundo.

## Anúncios

Há três espaços: uma faixa fixa no rodapé (320×50) e, em telas com 920 px ou
mais, uma coluna de cada lado (160×600). Para ligar, preencha o objeto `ADS` no
`index.html` com o `ca-pub-…` e o id de cada bloco criado no AdSense:

    var ADS = {
      client: "ca-pub-XXXXXXXXXXXXXXXX",
      slots: { bottom: "1234567890", left: "…", right: "…" },
      placeholder: true
    };

Sem `client`, os espaços aparecem só como moldura "Anúncio", para ver o layout;
`placeholder: false` esconde tudo até os anúncios existirem. O AdSense só
serve anúncio em domínio aprovado (não funciona abrindo o arquivo do disco), e
precisa de um `ads.txt` na raiz do site. Empacotado como app de loja, o caminho
é AdMob, não AdSense.

## Regerar os áudios

Falas em `falas.py`, geração (ElevenLabs) em `gerar_audios.py`:

    export ELEVENLABS_API_KEY="sk_..."
    python3 gerar_audios.py --teste          # 1 clipe por persona, ~200 créditos
    python3 gerar_audios.py                  # gera o que falta em out/
    python3 gerar_audios.py --dry-run        # só o custo

Para trocar uma fala: edite o texto, apague o `.mp3` correspondente e rode de
novo — só o apagado é refeito. Depois de copiar os clipes bons para `audios/`,
atualize o manifesto que a página consome:

    python3 gerar_audios.py --manifesto --out audios

## Classificação etária

A pasta `bar/` tem palavrão pesado. Se ela for junto num app de loja, a
classificação vai para 17+ (App Store) e 18 (Google Play) mesmo vindo desligada
— as lojas classificam pelo conteúdo presente, não pelo ativado. Publicando só
com `bar_leve/`, a classificação abre. Como web app isso não se aplica.
