# CalculadoraDeRepuxo

## Funcionalidade do aplicativo

O projeto a seguir demonstra um aplicativo de Windows escrito na linguagem de programação C#, em conjunto com a estrutura de interface de usuário Windows Forms.

O objetivo deste projeto é criar uma calculadora que receba do usuário os dados dimensionais de uma peça cilíndrica com aba, e calcule automaticamente todos os parâmetros que são relevantes na fabricação desta peça pelo processo de estampagem de repuxo.

A Figura a seguir mostra a tela inicial do aplicativo:

![](Imagens/Tela%20inicial%20vazia.png)

Com os dados dimensionais inseridos, o usuário pode apertar o botão `Calcular`. Este botão vai computar os dados inseridos e calcular o número de operações necessárias para a fabricação da peça, o volume da peça, o diâmetro do blank e o diâmetro interno de cada uma das operações, como visto no gif a seguir:

(Clique na imagem abaixo para ver o gif.)

![](Imagens/Tela%20inicial%20botao%20calcular.gif)

Feito isso, o usuário pode sair da página inicial e ir para a página `Calculadora`, mostrada na figura a seguir:

![](Imagens/Pagina%20Calculadora%20.png)

Esta página mostra um campo de seleção chamado `Operação:` onde o usuário pode escolher uma das operações de repuxo calculadas na página anterior, ao fazer isso, o aplicativo gera automaticamente todos os cálculos relacionados com a operação selecionada.

Este processo é demonstrado no gif a seguir:

(Clique na imagem abaixo para ver o gif.)

![](Imagens/Calculadora%20selecao%20da%20operacao.gif)

O aplicativo também permite que o usuário escolha manualmente o diâmetro interno inicial e final da operação, para isso basta marcar a opção `Manual` no programa, esta opção é útil para analisar parâmetros importantes na fabricação, por exemplo para saber até que ponto se pode diminuir o diâmetro interno sem que a severidade ultrapasse a severidade máxima, e como essa mudança afeta a força de repuxo e capacidade da prensa. 

Este processo é demonstrado no gif a seguir:

(Clique na imagem abaixo para ver o gif.)

![](Imagens/Calculadora%20modo%20manual.gif)

A maior vantagem deste programa é a praticidade na hora de planejar a fabricação deste tipo de peça, em pouco tempo o usuário pode analisar diversos casos diferentes e tomar decisões melhores em seu projeto.

(Clique na imagem abaixo para ver o gif.)

![](Imagens/Varias%20opcoes%20de%20peca.gif)

## Funcionalidade da programação

O programa possuí uma classe estática chamada `Dobra`, que é responsável por manter todos os métodos que efetuam os cálculos relacionados ao repuxo, como o cálculo do volume da peça, severidade, diâmetro do blank, etc.

A classe Dobra possui uma variável estática do tipo dicionário chamada `operações`, que tem como Integer o tipo de suas chaves e List<double> o tipo de seus valores; neste dicionário a chave corresponde às operações de repuxo, já o valor é uma lista onde o índice 0 corresponde ao parâmetro `D` daquela operação, e o índice 1 corresponde ao parâmetro `d`.

Ao apertar o botão `Calcular`, o programa verifica se todos os dados inseridos são válidos para o uso, ou seja, apenas valores numéricos positivos. Depois ele aciona um método que preenche o dicionário `operações` com todos os dados  das operações de repuxo.

Veja abaixo alguns exemplos do código:

```C#
        public static double getVolPeca(double dAba, double dInt, double alturaH, double esp, double Rm, double Rp)
        {
            // Este método calcula o volume da peça.
            double dm = dInt + esp;
            double Rmm = Rm + (esp / 2);
            double Rmp = Rp + (esp / 2);
            double alturaCilindro = alturaH - Rm - Rp - esp - esp;
            double dBlank = Math.Sqrt(Math.Pow(dAba,2) + (4 * dm * (alturaCilindro + (0.57 * (Rmm + Rmp)))));
            double volPeca = ((Math.PI) * Math.Pow(dBlank, 2) / 4) * esp;
            return Math.Round(volPeca,3);
        }
```
```C#
        public static int getNumOps(double dInt, double dBlank)
        {
            // Este método calcula o número de operações de repuxo.
            int count = 1;
            if (dInt >= dBlank)
            {
                return 1;
            }
            else
            {
                while (dBlank > dInt)
                {
                    if (count == 1)
                    {
                        dBlank = Math.Round(dBlank * 0.6, 3);
                        count++;
                    }
                    dBlank = Math.Round(dBlank * 0.8, 3);
                    count++;
                }
                return count - 1;
            }
        }
```


```C#
        public static void fillOperacoes(double dInt, double dBlank)
        {
            // Este método preenche a variável "operacoes" com os dados de cada operação.
            int numOps = getNumOps(dInt, dBlank);
            int countB = 0;
            int countC = numOps;
            while (countB < numOps)
            {
                if (countC == 1)
                {
                    operacoes.Add(countC, new List<double> { dInt, dBlank });
                    countB++;
                    countC--;
                }
                else
                {
                    dInt = Math.Round(dInt / 0.8, 3);
                    operacoes.Add(countC, new List<double> { Math.Round(dInt * 0.8, 3), dInt });
                    countB++;
                    countC--;
                }
            }
        }
```
```C#
        public static bool failedToConvertTxtBox(params TextBox[] textBox)
        {
            // Este método verifica se todos os inputs são válidos ou não para serem convertidos para double
            foreach (TextBox i in textBox)
            {
                try
                {
                    double a = Convert.ToDouble(i.Text);
                }
                catch
                {
                    return true;
                }
            }
            return false;
        }
```

Na página `calculadora` seleção é feita por um controle do tipo ComboBox, que é preenchido automaticamente com todas as chaves do dicionário `operações` da classe Dobra; ao selecionar algum valor do ComboBox é acionado um evento que pega os valores correspondes àquela chave e preenche os campos `D` e `d` abaixo da seleção.

```C#
                // Parte do código que preenche o ComboBox com as operações.
                int numOps = Dobra.getNumOps(Dint, Dblank);
                Dobra.fillOperacoes(Dint, Dblank);

                for (int i = 1; i < numOps + 1; i++)
                {
                    UC_Calculadora1.ucCalc1.cbbOperacoes.Items.Add(Convert.ToString(i));
                }
```
