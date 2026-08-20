# Como o simulador de investimentos da ArenaCash decide o que indicar

Análise do arquivo `simulador.js` do repositório [Product-Arena/arena-cash](https://github.com/Product-Arena/arena-cash).

## 1. O que o simulador faz

O simulador é a peça que decide, automaticamente, qual produto financeiro a ArenaCash sugere pra cada pessoa. Ele pede três informações no formulário — idade, salário mensal e o perfil que a pessoa mesma escolhe pra se descrever (conservador, moderado ou arrojado) — e devolve um produto específico, um valor sugerido de quanto investir por mês e uma lista de motivos explicando a escolha. Não tem inteligência artificial nem nada parecido: é uma sequência de regras fixas, escritas por alguém, que sempre dão o mesmo resultado pra uma mesma combinação de idade, salário e perfil.

## 2. Os produtos que ele pode indicar

| Produto | Rendimento | Prazo | Risco |
|---|---|---|---|
| CDB Reserva | 102% do CDI | Liquidez diária (sai quando quiser) | Muito baixo |
| LCI Isenta | 95% do CDI, sem Imposto de Renda | 1 ano de carência | Baixo |
| CDB Progressivo | 112% do CDI | 2 anos | Baixo |
| CDB Longo Prazo | 124% do CDI | 5 anos | Médio |
| Arena Quant | Variável, sem garantia | Resgate em 30 dias | Alto |

*CDI é a taxa que serve de referência para a maioria dos investimentos de renda fixa no Brasil — quando um produto rende "112% do CDI", significa que ele paga um pouco mais do que essa taxa de referência.*

## 3. Como a pontuação é calculada, passo a passo

1. **Calcula quanto a pessoa consegue investir por mês.** É sempre 15% do salário informado. Quem ganha R$ 3.000 por mês, por exemplo, teria um aporte sugerido de R$ 450.

2. **Transforma a idade em "tempo disponível".** A lógica é: quanto mais nova a pessoa, mais tempo ela tem antes de provavelmente precisar do dinheiro, e mais risco ela pode se dar ao luxo de correr.
   - Menos de 30 anos → 3 pontos
   - De 30 a 49 anos → 2 pontos
   - 50 anos ou mais → 1 ponto

3. **Transforma o perfil que a pessoa escolheu em pontos de tolerância a risco.**
   - Arrojado → 3 pontos
   - Moderado → 2 pontos
   - Conservador → 1 ponto

4. **Soma os dois números.** O resultado vai de 2 (o mínimo, alguém mais velho e conservador) a 6 (o máximo, alguém jovem e arrojado). Essa soma é o que decide o produto:

| Pontuação | Produto indicado |
|---|---|
| 2 | CDB Reserva |
| 3 | LCI Isenta |
| 4 | CDB Progressivo |
| 5 | CDB Longo Prazo |
| 6 | Arena Quant (com uma condição extra — veja a seção 4) |

## 4. As regras que passam na frente da pontuação

Duas regras no arquivo têm prioridade sobre a conta da seção anterior — elas mudam o resultado mesmo que a pontuação diga outra coisa:

- **Reserva de emergência antes de qualquer coisa.** Se o valor que a pessoa consegue investir por mês for menor que R$ 100, a indicação é sempre o CDB Reserva — não importa a idade, não importa o perfil. A ideia é que quem ainda não consegue guardar nem R$ 100 por mês não deveria travar esse dinheiro num produto de prazo longo, mesmo que se considere arrojado.

- **Freio no produto mais arriscado.** Mesmo quando a pontuação bate o máximo (6, jovem e arrojado), o Arena Quant só é indicado se a pessoa conseguir investir pelo menos R$ 500 por mês. Se não conseguir, a indicação desce um degrau e vira o CDB Longo Prazo.

## 5. Casos-limite

Situações em que um pequeno detalhe muda completamente o resultado — e onde a explicação que a tela mostra pode não parecer suficiente pra quem recebeu a indicação.

**Faltar pouco pra reserva mínima**
Idade 24, salário R$ 650, perfil arrojado → aporte de R$ 97,50 (arredonda pra R$ 98), abaixo do mínimo de R$ 100 → recebe o CDB Reserva, o produto mais conservador, mesmo sendo jovem e se dizendo arrojado.

**Faltar pouco pro Arena Quant**
Idade 24, salário R$ 3.300, perfil arrojado → pontuação máxima (6 de 6), mas aporte de R$ 495 — R$ 5 abaixo do mínimo de R$ 500 exigido pro Arena Quant → recebe o CDB Longo Prazo em vez do produto mais arriscado. Um salário R$ 34 maior já mudaria o resultado.

**Rendimento menor pra quem "arriscou mais"**
Idade 55, perfil conservador → pontuação 2 → CDB Reserva (102% do CDI).
Idade 35, perfil conservador → pontuação 3 → LCI Isenta (95% do CDI).
Quem teve pontuação de risco mais alta recebeu um produto cujo rendimento anunciado é um número menor — porque a isenção de Imposto de Renda da LCI compensa na prática, mas isso não aparece explicado lado a lado com o percentual.

**Idade pesando mais do que o perfil escolhido**
Idade 65, perfil arrojado → pontuação 4 → CDB Progressivo (risco baixo).
Idade 25, perfil moderado → pontuação 5 → CDB Longo Prazo (risco médio).
Quem se descreveu como arrojado recebeu um produto de risco baixo, e quem se disse apenas moderado recebeu um de risco médio — porque a idade conta tantos pontos quanto a resposta que a pessoa deu sobre si mesma.

## 6. Perguntas que eu levaria pro time de produto

1. A regra da reserva mínima (R$ 100) é um corte seco: R$ 98 e R$ 102 dão indicações opostas. Faz sentido não ter nenhuma transição entre "reserva obrigatória" e "pontuação normal"?

2. O limite de R$ 500 pro Arena Quant também é um corte seco, e pode barrar justamente quem teve a pontuação de risco máxima. Esse valor foi testado com clientes reais, ou é uma estimativa?

3. A LCI é indicada com um rendimento nominal mais baixo (95% do CDI) do que a opção mais conservadora (102% do CDI). O cliente vê isso lado a lado na tela? Se sim, existe alguma explicação da isenção de IR junto, pra não parecer um erro?

4. Faz sentido que a idade pese exatamente o mesmo que a resposta que a pessoa dá sobre o próprio perfil? Um cliente de 65 anos "arrojado" recebendo um produto de risco mais baixo que um de 25 anos "moderado" é intencional?

5. Existe algum teste com usuários reais mostrando se essas indicações batem com o que as pessoas esperavam receber, principalmente nos casos-limite acima?

6. Quando a indicação "desce um degrau" (por causa do aporte mínimo do Arena Quant, por exemplo), a tela explica isso claramente, ou a pessoa só vê o produto final sem entender por que não recebeu o que a pontuação sugeria?
