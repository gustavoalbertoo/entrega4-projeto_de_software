# Minha Refeição – Serviço de Assinatura de Marmitas

**Entrega 4 – Implementação em Java do caso de uso "Assinar Plano de Refeições"**
Disciplina: Projeto de Software – Prof.ª Ana Claudia Rossi – Ciência da Computação (Mackenzie)
Grupo: Gustavo Rodrigues Alberto (10738010) e Gabriel Mello Aristides (10736402)

Projeto organizado em **uma fronteira** (`ViewAssinatura`), **um controle** (`CtlAssinarPlano`) e as
**entidades** (pacote `entidade`). Aplicação de **terminal** que implementa o cenário modelado nos diagramas de sequência DS1 a DS5 e no
Diagrama de Classes de Projeto: fluxo principal e fluxos alternativos **FA1** (código inválido),
**FA2** (quantidade incompatível com o plano) e **FA3** (pagamento não autorizado).

Além do enunciado, a implementação inclui duas regras de negócio: **cadastro completo com recusa de
celular/e-mail duplicados** (FA4) e **confirmação do resumo do pedido**, com retorno à escolha do plano
quando não confirmado (FA5). Veja os itens 5 e 6.

---

## 1. Requisitos

- **JDK 17 ou superior** (testado com JDK 21). Confira com `java -version` e `javac -version`.
- **VS Code** com o *Extension Pack for Java* (recomendado), **ou** apenas um terminal.

Não há bibliotecas externas nem Maven/Gradle.

## 2. Estrutura do projeto

```
minha-refeicao/
├── dados/                      arquivos CSV (entrada e saída de dados)
│   ├── planos.csv              planos de assinatura (entrada)
│   └── cardapio.csv            itens do cardápio (entrada)
└── src/
    ├── Main.java               ponto de entrada
    ├── fronteira/              ViewAssinatura                      (única fronteira)
    ├── controle/               CtlAssinarPlano                     (único controle)
    └── entidade/               demais classes (domínio)
        ├── Assinante, Assinatura, PlanoAssinatura, Pedido, ItemPedido, EnderecoEntrega
        ├── Cardapio, ItemCardapio (abstrata) com PratoPrincipal / Acompanhamento / Sobremesa
        ├── PreferenciaAlimentar, CodigoConfirmacao, CartaoCredito, Pagamento
        ├── OperadoraCartaoCredito, RespostaAutorizacao
        └── Collections e arquivos CSV: ColecaoPlanos, ColecaoItensCardapio, ColecaoAssinaturas, ArquivoCsv
```

## 3. Como executar

> **Importante:** abra no VS Code (ou entre no terminal) a pasta do projeto, **a que contém `src` e `dados`**.
> O programa mostra na primeira linha de onde carregou os dados: `Dados carregados de: ...\dados`.

### Opção A – VS Code (recomendada)

1. `File > Open Folder...` e escolha a pasta do projeto.
2. Abra o arquivo `src/Main.java`.
3. Clique em **Run** (botão ▶ acima do `main`, da extensão *Java*). O programa roda no **terminal integrado**,
   onde você digita as respostas.

*Usando a extensão Code Runner:* ative `File > Preferences > Settings`, procure por
**"Code-runner: Run In Terminal"** e marque a opção. Sem isso, o painel *Output* não aceita digitação.

### Opção B – Terminal (Windows PowerShell, Linux ou macOS)

Na pasta do projeto (onde estão `src` e `dados`):

```
javac -d out -sourcepath src src/Main.java
java -cp out Main
```

Alternativa, de dentro da pasta `src`:

```
cd src
javac Main.java
java Main
```

Se aparecer `ERRO: pasta 'dados' ... nao encontrada`, você não está na pasta do projeto: use `cd` até a pasta
que contém `src` e `dados`.

## 4. Simulações (sistemas externos)

| Sistema externo | Como foi simulado |
|---|---|
| Envio de SMS (`CodigoConfirmacao.enviarSMS()`) | O código de 6 dígitos aparece no próprio terminal: `[SMS para ...] ... codigo ... e 123456`. Basta digitá-lo. |
| Operadora de cartão (`OperadoraCartaoCredito.solicitarAutorizacao()`) | Cartão cujo **último dígito é PAR → aprovado**; **ÍMPAR → recusado**. |

Cartões de teste: `4111 1111 1111 1112` (**aprova**) e `4111 1111 1111 1111` (**recusa**).
Validade: qualquer data futura no formato `MM/AA` (ex.: `12/30`).

## 5. Roteiro de teste de cada fluxo

Os números dos itens (`id`) dependem das preferências escolhidas: use sempre os mostrados na tela.
Os exemplos abaixo usam o **Plano 1 (Semanal, 5 refeições)** e as preferências **2,3 (Vegetariana + Sem lactose)**.

> **Atenção ao repetir os testes:** celular e e-mail são **únicos**. Depois que uma assinatura fica **Ativa**,
> usar os mesmos dados de novo é recusado (FA4). Para repetir uma demonstração, use outro celular/e-mail ou
> apague o arquivo `dados/assinantes.csv`.

### Fluxo principal
| Pergunta | Resposta de exemplo |
|---|---|
| Nome completo | `Gustavo Alberto` |
| Gênero | `2` (1-Feminino, 2-Masculino, 3-Outro, 4-Prefiro não informar) |
| E-mail | `gustavo@email.com` |
| Celular (DDD + número) | `11987654321` |
| Código recebido por SMS | o código exibido no terminal |
| Plano / confirma | `1` / `S` |
| Preferências | `2,3` |
| Pratos principais (total exato = 5) | `6:3,8:2` |
| Acompanhamentos (opcional, até 5) | `9:5` |
| Sobremesas (opcional, até 5) | `17:4,20` |
| Confirma o pedido? | `S` |
| Endereço | `01302-907`, `Rua da Consolacao`, `930`, complemento opcional, `Consolacao`, `Sao Paulo`, `SP`, confirma `S` |
| Cartão | `4111 1111 1111 1112`, nome, `12/30`, bandeira `1` |

Resultado: pagamento aprovado, assinatura **Ativa**, pedido **Aprovado**, protocolo e previsão da primeira entrega.
O assinante passa a constar em `dados/assinantes.csv`.

### FA1 – Código de confirmação inválido
Digite um código errado **3 vezes**. O sistema encerra a validação e pede um novo código:
`ENTER` reenvia o SMS ao mesmo celular, ou `N` volta ao cadastro para corrigir os dados. Com o código correto, o fluxo segue para os planos.

### FA2 – Quantidade incompatível com o plano
No Plano 1, nos pratos principais informe mais de 5 itens (ex.: `6:3,8:4`). O sistema informa o limite do plano
e pede nova seleção. O mesmo vale para acompanhamentos e sobremesas.

### FA4 – Cadastro duplicado (extensão)
Cadastre um assinante e conclua a assinatura. Em uma nova execução, informe o **mesmo celular** ou o **mesmo
e-mail** (maiúsculas/minúsculas são ignoradas): o sistema mostra `CADASTRO RECUSADO` e pede os dados de novo,
antes de enviar o SMS.

### FA5 – Pedido não confirmado (extensão)
No **RESUMO DO PEDIDO**, responda `N` em "Confirma o pedido?". O pedido é descartado e o sistema volta à
**lista de planos** (o cadastro já validado é mantido): escolha plano, preferências e itens novamente.

### FA3 – Pagamento não autorizado
Informe o cartão `4111 1111 1111 1111` (final ímpar). O sistema avisa que o pagamento não foi aprovado e a
assinatura permanece **Aguardando Pagamento**. Então:
- responda `S` em "Deseja informar outro cartão?" e use `4111 1111 1111 1112` → pagamento aprovado e fluxo principal continua; ou
- responda `N` → processo encerrado **sem ativação** (assinatura gravada como **Cancelada**).

## 6. Regras de negócio implementadas

- O cadastro exige **nome, gênero, e-mail e celular**. Celular e e-mail são **únicos**: se já pertencerem a um
  assinante, o cadastro é recusado (FA4). O assinante só é gravado em `assinantes.csv` quando a assinatura
  fica **Ativa**; quem desiste após recusa de pagamento (Cancelada) pode tentar novamente com os mesmos dados.
- O código SMS tem **3 tentativas**; ao esgotá-las é preciso solicitar novo código (FA1).
- O resumo do pedido precisa ser **confirmado**; caso contrário o pedido é descartado e volta-se à escolha do plano (FA5).
- O limite do plano vale **por categoria**: num plano de N refeições, o assinante escolhe **exatamente N pratos
  principais** e **até N** acompanhamentos e **até N** sobremesas (acima disso ocorre o FA2).
- Com várias preferências, só aparecem os itens que atendem a **todas** elas (a opção *Tradicional* não restringe).
- O valor total da assinatura é o valor do plano.
- A previsão da primeira entrega é a data da contratação + 2 dias (Semanal), 3 (Quinzenal) ou 5 (Mensal).
- Status da assinatura: *Em Elaboração* → *Aguardando Pagamento* → *Ativa* (ou *Cancelada* se o assinante desistir após recusa).

## 7. Persistência (arquivos CSV, separador `;`)

| Arquivo | Tipo | Conteúdo |
|---|---|---|
| `dados/planos.csv` | entrada | planos disponíveis |
| `dados/cardapio.csv` | entrada | itens do cardápio (coluna `compativelCom` indica Vegetariana / Sem lactose) |
| `dados/assinantes.csv` | saída (criado ao executar) | celular, nome, gênero, e-mail e preferências dos assinantes com assinatura Ativa |
| `dados/assinaturas.csv` | saída | assinatura, status, protocolo, valor e endereço |
| `dados/itens_pedido.csv` | saída | itens de cada pedido |
| `dados/pagamentos.csv` | saída | **todas** as tentativas de pagamento (Aprovado / Recusado) |

Os arquivos de saída são criados automaticamente quando uma assinatura termina (Ativa ou Cancelada).
Do cartão é gravado apenas o final (4 últimos dígitos). Para "zerar" os dados, apague os quatro arquivos de saída. Se existir um `assinantes.csv` de versão anterior
(outro formato), ele é preservado automaticamente como `assinantes.csv.antigo`.

## 8. Correspondência com a modelagem

| Elemento do diagrama | Onde está no código |
|---|---|
| `viewAssinatura` (única fronteira) | pacote `fronteira` |
| `ctlAssinarPlano` (único controle) | pacote `controle` |
| Entidades e *collections* (`Assinatura`, `Pedido`, `colecaoPlanos`, `colecaoAssinaturas`...) | pacote `entidade` |
| Envio do SMS e autorização do cartão | `CodigoConfirmacao.enviarSMS()` e `OperadoraCartaoCredito.solicitarAutorizacao()` (entidades) |
| Mensagens do diagrama de sequência | métodos de mesmo nome (ex.: `validarCodigo`, `selecionarItens`, `efetuarPagamento`) |
| Persistência em CSV | `ColecaoPlanos`, `ColecaoItensCardapio` e `ColecaoAssinaturas` (usam `ArquivoCsv`) e a pasta `dados` |

A fronteira só se comunica com o controle; o controle chama as entidades.

Cada classe e método está comentado indicando a mensagem do diagrama correspondente.
