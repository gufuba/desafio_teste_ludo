# Desafio de Teste — Ludo

Casos de teste em **JavaScript (Jest)** para uma **partida de Ludo para 2 jogadores**.
Desenvolvido com TDD: primeiro os testes (commit `test:`, falhando), depois a implementação
(commit `feat:`).

## Colaborador

- **Gustavo Fumagalli** — [@gufuba](https://github.com/gufuba)

## Especificação do desafio

- Casos de teste para testar **uma partida para 02 jogadores**.
- Casos de teste em **JavaScript** para testar **JavaScript**.

### Regras implementadas

| Item | Decisão |
|---|---|
| Jogadores e peças | 2 jogadores (1 e 2), **4 peças** cada, começando na **base**; o jogador 1 joga primeiro |
| Caminho | Circuito de **52 casas**; o jogador 1 entra na casa **0** e o jogador 2 na casa **26** (lados opostos) |
| Sair da base | Só com **6**; a peça vai para a casa inicial do jogador |
| Jogar de novo | Tirar **6** dá direito a jogar novamente (depois de mover a peça) |
| Sem jogada | Se nenhuma peça pode se mover com o dado, a vez passa ao adversário automaticamente |
| Captura | Cair em casa com peças adversárias as envia de volta para a **base** (todas as que estiverem na casa) |
| Casas seguras | Casas iniciais e estrelas (`0, 8, 13, 21, 26, 34, 39, 47`): **não há captura**, as peças dividem a casa |
| Reta final | Depois de 51 casas do caminho a peça entra em **5 casas exclusivas** (sem captura) e depois na chegada |
| Chegada | Exige o **número exato** do dado |
| Vitória | Vence quem levar as **4 peças** para a chegada |

**Progresso de uma peça:** `-1` = base · `0..50` = caminho · `51..55` = reta final · `56` = chegada.

## Como executar

```bash
npm install
npm test        # roda os testes com relatório de cobertura (exige 100%)
```

Requer Node.js 18 ou superior.

## Estrutura

```text
src/ludo.js            # classe Ludo e constantes do tabuleiro
tests/ludo.test.js     # casos de teste (Jest)
tests/helpers.js       # gerador pseudoaleatório com semente
```

### API usada pelos testes

```js
const jogo = new Ludo({ dado });          // dado: função que devolve 1..6 (padrão: aleatório)
jogo.rolarDado();                         // rola o dado (sem jogada possível, a vez passa)
jogo.movimentosValidos();                 // índices (0..3) das peças que podem se mover
jogo.mover(indice);                       // { peca, de, para, capturadas }
jogo.pecas(jogador);  jogo.estadoDaPeca(jogador, i);  jogo.posicaoGlobal(jogador, i);
jogo.estado;   // 'jogando' | 'fim'
jogo.vez;  jogo.vencedor;  jogo.dadoAtual;  jogo.ultimoDado;

new Ludo({ situacao: { pecas: [[...4 progressos], [...4 progressos]], vez: 1 } });  // retoma um cenário
```

O dado é **injetável**, o que torna os testes determinísticos (`dado: () => 6`).

## Funcionamento dos casos de teste

66 testes, organizados em grupos (`describe`):

| Grupo | O que verifica |
|---|---|
| **O tabuleiro e as peças** | 52 casas, casas iniciais 0 e 26, casas seguras; início com 4 peças na base e jogador 1 na vez; estados da peça (`base`, `caminho`, `reta-final`, `chegada`); conversão do progresso em casa do caminho (inclusive "dar a volta"); rejeição de jogador, peça e situação inválidos. |
| **Rolar o dado** | Devolve e registra o valor; rejeita valores inválidos (`0`, `7`, `1.5`, texto, `undefined`); dado padrão sempre de 1 a 6; não rola duas vezes sem mover; não move sem rolar. |
| **Sair da base (só com 6)** | Sem 6 e sem peças no caminho a vez passa; com 6 qualquer peça da base sai; vai para a casa inicial de cada jogador (0 e 26); sem 6, só peças do caminho andam. |
| **Mover pelo caminho** | Avança o valor do dado e passa a vez; **6 dá nova jogada**; o jogador escolhe a peça; o jogador 2 dá a volta no tabuleiro; rejeita índice inválido e peça que não pode se mover. |
| **Captura** | Captura uma e várias peças; vale para os dois jogadores; **7 casas seguras** (+ a casa inicial) não permitem captura; não captura peça do próprio jogador nem peça na reta final. |
| **Reta final e chegada** | Entra na reta final depois da casa 50; chega com número exato; dado maior que o necessário não move a peça; sem peça movível a vez passa; peça que chegou não se move mais. |
| **Fim da partida** | Vitória do jogador 1 e do jogador 2; vitória com um 6 encerra o jogo; com 3 peças na chegada o jogo continua; nada é permitido depois do fim. |
| **Partidas completas** | 40 partidas automáticas (dados com semente) terminam com o vencedor com as 4 peças em `56`; a cada lance, os progressos ficam entre `-1` e `56` e **fora das casas seguras nunca há peças de jogadores diferentes na mesma casa**; a mesma sequência de dados gera a mesma partida. |

**Cobertura:** `jest.coverageThreshold` exige **100 %** de instruções, ramos, funções e linhas;
o `npm test` falha se a cobertura cair. O workflow do GitHub Actions
(`.github/workflows/testes.yml`) roda os testes a cada push.
