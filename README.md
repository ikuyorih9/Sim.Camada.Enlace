# Camada-de-Enlace
Projeto implementado como segundo trabalho da disciplina Redes de Computadores (SSC0641), ministrada pelo professor Rodolfo Ipolito Meneguette.

## Hierarquia das camadas.

<p align="center">
    <img width=500 src="img/Figura - hierarquia camadas.png">
</p>

### X.1. Aplicação transmissora.

### X.2. Camada de aplicação transmissora.

### X.3. Camada de enlace de dados transmissora.

A **camada de enlace de dados transmissora** é chamada pela camada de aplicação transmissora. Ela começa realizando o *controle de erro para transmmissão* do quadro, que consiste na codificação do quadro original em um novo com bits anexados ao seu fim. A quantidade e o valor dos bits anexados depende do tipo de controle, que pode ser por ``paridade`` ou ``CRC``.

#### X.3.1. **Codificação da transmissão por paridade.**

O controle da transmissão por paridade consiste em *adicionar um único bit ao final do quadro*, funcionando como um booleano. 

O bit de paridade, anexado ao final do quadro, depende do tipo de codificação (par ou ímpar) e da quantidade de 1's. A tabela abaixo mostra que a relação entre esses dois parâmetros equivale a uma lógica XOR.

| Codificação | Quantidade de '1's | Bit de paridade |
|:-----------:|:------------------:|:---------------:|
|  ímpar (0)  |      ímpar (0)     |        0        |
|  ímpar (0)  |       par (1)      |        1        |
|   par (1)   |      ímpar (0)     |        1        |
|   par (1)   |       par (1)      |        0        |

##### Tabela 1: tabela-verdade do controle de paridade para transmissão.

Dessa forma, o algorítmo de codificação da transmissão por paridade é dado pelo código abaixo.

```
void controlaParidadeTransmissao(bool controlePar){
    //ALOCA MEMÓRIA PARA O QUADRO CODIFICADO.

    int tamanho = this->tamanho;
    int * quadro = new int[tamanho + 1];
    memcpy(quadro, this->quadro, sizeof(int) * tamanho);

    //REALIZA A LÓGICA XOR PARA O BIT DE PARIDADE.

    quadro[tamanho] = (int)(controlePar ^ retornaSePar(this->quadro, tamanho));

    //ATUALIZA O QUADRO COM O QUADRO CODIFICADO.

    delete [] this->quadro; //Libera memória alocada para o último quadro.
    this->quadro = quadro;
    this->tamanho = tamanho + 1;
}
```
##### Código 1: função de controle de paridade na transmissão.

Observe que o booleano `controlePar` corresponde ao "Codificação" da tabela, enquanto a função `retornaSePar()` obtém a informação da paridade de 1's.

#### X.3.2. **Codificação da transmissão por CRC.**

O controle da transmissão por CRC consiste em transformar o valor do quadro em um valor divisível por um **polinômio gerador**. Como o quadro é binário, o polinômio gerador é convertido em um número binário considerando os seus coeficientes.

O polinômio gerador utilizado é escolhido conforme o padrão IEE 802.3, ou seja

> x<sup>31</sup> + x<sup>25</sup> + x<sup>22</sup> + x<sup>21</sup> + x<sup>15</sup> + x<sup>11</sup> + x<sup>10</sup> + x<sup>9</sup> + x<sup>7</sup> + x<sup>6</sup> + x<sup>4</sup> + x<sup>3</sup> + x<sup>1</sup> + x<sup>0</sup>

que, em binário se torna o número de 32 bits:

> 1000 0010 0110 0000 1000 1110 1101 1011

Para transformar o quadro em um número divisível por 1000 0010 0110 0000 1000 1110 1101 1011, é preciso:

1. Anexar 31 bits 0 ao final do quadro;
2. Realizar a divisão binária de módulo 2 (XOR) entre o quadro e o número 1000 0010 0110 0000 1000 1110 1101 1011;
3. Subtrair o quadro pelo resto da divisão.

Assim, o algorítimo dessa codificação se dá por

``` 
ANEXA 31 BITS ZERO À DIREITA DO QUADRO.

for(i = 0; i < this->tamanho; i++){
    quadro[i] = this->quadro[i];
}
for(; i<tamanho; i++){
    quadro[i] = 0;
}

//REALIZA A DIVISÃO DE MÓDULO 2.

int * resto = retornaRestoDivisao(quadro, tamanho, coeficientes, 32);

//SUBTRAI O RESTO DO QUADRO (XOR).

for(int i = 0; i < tamanho; i++){
    quadro[i] ^= resto[i];
}

```
##### Código 2: função de controle CRC na transmissão.


Por fim, a a camada de enlace de dados de transmissão chama o **meio de comunicação**.

### X.4. Meio de comunicação.

O **meio de comunicação** passa o quadro da camada de enlace transmissora para a camada de enlace receptora. É nesse ponto que pode ocorrer erros na mensagem. Assim, dada uma porcentagem limiar para erros `porcentagemErros`, a simulação do meio de comunicação gera um número aleatório para cada bit do quadro. Se o valor for maior que o limiar, o meio transmite corretamente o bit. Caso contrário, o meio inverte o bit.

```
//PARA CADA BIT DO QUADRO.

for(int i = 0; i < tamanho; i++){
    int porcentagemAleatoria = rand()%100 + 1;

    //VERIFICA SE O BIT É TRANSMITIDO CORRETAMENTE OU NÃO.
    
    if(porcentagemAleatoria >= porcentagemErros)
        fluxoBrutoBytesPontoB[i] = fluxoBrutoBytesPontoA[i];
    else
        fluxoBrutoBytesPontoB[i] = !fluxoBrutoBytesPontoA[i];
}
```

Após esse processo, a camada de enlace de dados receptora é chamada.

### X.5. Camada de enlace de dados receptora.

A **camada de enlace de dados receptora** obtém o quadro de dados binário codificado e o decodifica, através do controle de erro. Da mesma forma que na transmissão, a camada receptora faz o controle de **paridade** e **CRC**. 

#### X.4.1. Decodificação na recepção por paridade.

A decodificação por paridade é simples, basta contar a quantidade de bits 1 e comparar com o bit de paridade. Dessa forma, precisa-se verificar o tipo de **decodificação**, a **paridade** do quadro e o **bit de paridade**. A relação entre elas e o erro é dada por uma lógica XOR, conforme a tabela abaixo.

| **Decodificação** | **Quantidade de '1's** | **Bit de paridade** | **Erro** |
|:-----------------:|:------------:|:-------------------:|:--------:|
|      ímpar(0)     |   ímpar(0)   |       ímpar(0)      |     0    |
|     ímpar (0)     |   ímpar (0)  |       par (1)       |     1    |
|     ímpar (0)     |    par (1)   |      ímpar (0)      |     1    |
|     ímpar (0)     |    par (1)   |       par (1)       |     0    |
|      par (1)      |   ímpar (0)  |      ímpar (0)      |     1    |
|      par (1)      |   ímpar (0)  |       par (1)       |     0    |
|      par (1)      |    par (1)   |      ímpar (0)      |     0    |
|      par (1)      |    par (1)   |       par (1)       |     1    |

##### Tabela 2: tabela-verdade do controle de paridade para recepção.

O código da decodificação por paridade é dado por:
```
void CamadaEnlace::controlaParidadeRecepcao(bool controlePar){
    //ALOCA MEMÓRIA PARA O QUADRO DECODIFICADO.

    int tamanho = this->tamanho;
    int * quadro = new int[tamanho-1];
    memcpy(quadro, this->quadro, sizeof(int) * (tamanho - 1));

    //OBTÉM A PARIDADE DO QUADRO E O BIT DE PARIDADE.

    bool paridade = retornaSePar(quadro, tamanho-1); //Retorna se a quantidade de 1's é par (true) ou ímpar (false).

    bool bitParidade = (bool) this->quadro[tamanho - 1]; //Obtém o último bit do quadro, indicador de paridade.

    //VERIFICA ERRO DE PARIDADE.

    bool erroQuadro = (bool)(controlePar ^ paridade ^ bitParidade);

    if(erroQuadro)
        exit(-1);

    //ATUALIZA O QUADRO COM O QUADRO DECODIFICADO.

    delete [] this->quadro;
    this->quadro = quadro;
    this->tamanho = tamanho - 1;
}
```

#### X.4.2. Decodificação na recepção por CRC.

A **decodificação na recepção por CRC** acontece verificando se o quadro codificado é divisível, em módulo 2, pelo polinômio gerador (1000 0010 0110 0000 1000 1110 1101 1011). Se o resto da divisão é zero, o quadro não tem erro, sendo necessário apenas retirar os últimos 31 bits do quadro. Caso contrário, o quadro está com o erro.

Dessa forma, o algoritmo é dado por:

```
//OBTÉM O RESTO DA DIVISÃO DE MÓDULO 2 E VERIFICA SE NÃO É VAZIO (ERRO).

int * resto = retornaRestoDivisao(this->quadro, this->tamanho, coeficientes, 32);

if(!arrayBinarioEstaZerado(resto, this->tamanho))
    exit(-1);

//COPIA APENAS O QUADRO ORIGINAL, SEM OS 31 BITS.

int tamanho = this->tamanho - GRAU_COEF;
int * quadro = new (nothrow) int[tamanho];
memcpy(quadro, this->quadro, sizeof(int) * tamanho);

//ATUALIZA QUADRO COM QUADRO DECODIFICADO.

delete [] this->quadro; //Libera o quadro salvo.
this->quadro = quadro;
this->tamanho = tamanho;
```

### X.6. Camada de aplicação receptora.


### X.7. Aplicação receptora.