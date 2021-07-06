# Gerador, Iterável e Iterador como solução de alocação de memória utilizando lógica de programação lazy evaluation no Python.

>Livio Alvarenga - Jul/2021

&nbsp;

![Iterável Vs Iterador](imagens/Gerador_Iterador_Iteravel.png)

`Figura1 - Representação do fluxo do artigo.`

&nbsp;

O objetivo do artigo é demonstrar a importância de usarmos a lógica de programação avaliação preguiçosa ([lazy evaluation](https://pt.wikipedia.org/wiki/Avalia%C3%A7%C3%A3o_pregui%C3%A7osa)), trilhando o estudo pelo fluxo da figura 1.

---

Conteúdo:

1. [Os dados](#dados)
2. [Conteiner de dados](#conteiner)
   1. [getsizeof()](#gets)
3. [O método \_\_getitem__](#getitem)
4. [É iterável?](#iteravel)
   1. [Objeto iterável Vs objeto iterador](#iter_itera)
5. [Aplicando método \_\_iter__](#iter)
6. [Lógica avaliação preguiçosa (Lazy evaluation)](#lazy)
   1. [Criando uma classe aplicando Lazy evaluation](#class_lazy)
7. [Expressões geradoras](#expr_geradoras)
8. [Função Geradora (Aplicando yield)](#gerador)
9. [Função Geradora para ler linha a linha de um arquivo](#solucao)
10. [Conclusão](#conclusão)

---

&nbsp;

<div id='dados'/>

## 1 - Dados

&nbsp;

Temos listas [ ], tuplas ( ), conjuntos { }, dicionários {chave, valor}, arquivos etc. Todos esses itens podem ser considerados objetos no Python. Por sua vez, podemos atribuir dados nos mesmos e assim produzir conteiners de dados.


---

&nbsp;

<div id='conteiner'/>

## 2 - Conteiner de dados

&nbsp;

Usaremos o arquivo [VendasTesouroDireto.csv](https://dados.gov.br/dataset/vendas-do-tesouro-direto1). O mesmo traz o volume de vendas diária do programa Tesouro Direto. Segue cabeçalho e a primeira linha dessa base de dados. 

&nbsp;

Tipo Titulo | Vencimento do Titulo | Data Venda | PU | Quantidade | Valor
:---: | :---: | :---: | :---: | :---: | :---: |
Tesouro IPCA+ com Juros Semestrais | 15/05/2011 | 22/09/2008 | 1678,137281 | 664,40 | 1114954,40

&nbsp;

Ela será nosso objeto Iterável. Temos 65.760 linhas. Nosso objetivo é, independentemente do seu tamanho, aplicar um iterador e a lógica de programação avaliação preguiçosa ([lazy evaluation](https://pt.wikipedia.org/wiki/Avalia%C3%A7%C3%A3o_pregui%C3%A7osa)). 

Vamos fazer isso de duas formas: uma criando uma classe com os métodos \_\_iter__ e \_\_next__ e a outra usando geradores. Durante toda aplicação vamos medir o uso de memória em nossas variáveis utilizando a função getsizeof( ).

---

<div id='gets'/>

### 2.1 - getsizeof()

&nbsp;

getsizeof() retorna o tamanho de um objeto em bytes.

~~~python
from sys import getsizeof

x = 5
y = [1, 'Python']

print(f'x = {getsizeof(x)} bytes')
print(f'y = {getsizeof(y)} bytes')
~~~

`Retorna:`

```
x = 28 bytes
y = 72 bytes
```

Vamos verificar de que tamanho seria uma variável com o arquivo 

~~~python
# Verificando o tamanho do arquivo VendasTesouroDireto.csv
with open('VendasTesouroDireto.csv', 'r', encoding="utf-8") as a:
    conteúdo = a.read()

print(f'VendasTesouroDireto.csv = {getsizeof(conteúdo)} bytes')
print(f'VendasTesouroDireto.csv = {getsizeof(conteúdo)/1000} Kbytes')
~~~

`Retorna:`

```
VendasTesouroDireto.csv = 5010412 bytes
VendasTesouroDireto.csv = 5010.412 Kbytes
```
Estaríamos com a variável _conteúdo_ consumindo 5010.412 Kbytes.


---

&nbsp;

<div id='getitem'/>

## 3 - O método \_\_getitem__

&nbsp;

Para confirmar que um objeto pode receber o método \_\_getitem__ usaremos a função dir().

dir() retorna uma lista de strings com todos os métodos que podem ser usados no objeto. Assim confirmamos que o método \_\_getitem__ é um método aplicável ao objeto conteúdo.

~~~python
# Confirmando que conteudo tem o método __getitem__.
'__getitem__' in dir(conteudo)
~~~

`Retorna:`

```
True
```

Para entendermos sobre o uso de memória vamos passar pela aplicação do método \_\_getitem__. Este é o que da condição para fazermos slice com objetos.

~~~python
# fazendo slice em nosso objeto
print(conteudo[0:151])
print(conteudo[1000:1300])
~~~

`Retorna:`

```
Tipo Titulo;Vencimento do Titulo;Data Venda;PU;Quantidade;Valor
Tesouro IPCA+ com Juros Semestrais;15/05/2011;22/09/2008;1678,137281;664,40;1114954,40

 Prefixado com Juros Semestrais;01/01/2017;17/11/2008;730,230014;410,80;299978,48
Tesouro IPCA+;15/05/2015;17/11/2008;966,989586;750,00;725242,18
Tesouro Selic;07/03/2012;17/11/2008;3668,550000;239,00;876783,45
Tesouro Prefixado com Juros Semestrais;01/01/2014;17/11/2008;801,760094;658,80;528199,54
```
Um ponto interessante que irá ajudar-nos no raciocínio mais a frente é que com o método \_\_getitem__ conseguimos acessar nosso objeto nos primeiros 150 caracteres e novamente acessá-lo nas posições de 1000:1300. Isso só é possível por estarmos com o objeto todo alocado na memória. 

E se esse arquivo, com o passar dos tempos, chegar a 5, 10, 20 gigabytes? 

Seria inviável manipular esse objeto dessa forma. Guardem este conceito, pois vamos começar a criar uma forma de acessar objetos iteráveis de qualquer tamanho, usando a memória somente para nossas manipulações.

---

&nbsp;

<div id='iteravel'/>

## 4 - É iterável?

&nbsp;

Vamos confirmar que o método \_\_iter__ é um método aplicável ao objeto conteúdo.

~~~python
# Confirmando que conteúdo tem o método __iter__ de iterável.
'__iter__' in dir(conteúdo)
~~~

`Retorna:`

```
True
```
---
<div id='iter_itera'/>

### 4.1 - Objeto iterável Vs objeto iterador

&nbsp;

Já concluímos que nosso objeto é um **Iterável**. Fizemos isso aplicando o código (`'__iter__' in dir(conteúdo)`). Agora vamos verificar se nosso objeto pode receber o método \_\_next__. Se sim, ele também é um iterador.

~~~python
# Confirmando se conteudo tem o método __next__ de iterador.
'__next__' in dir(conteudo)
~~~

`Retorna:`

```
False
```
Nosso objeto pode receber o método \_\_iter__, ou seja, confirmamos que ele é um **ITERÁVEL**. Mas, para nossa surpresa, notamos que ele **NÃO É UM ITERADOR**, pois ele não pode receber o método \_\_next__.

Esse conceito tem que estar claro em nossa mente: o que é um iterável e o que é um iterador, pelo menos como identifica-los. Vamos continuar seguindo o fluxo da figura1 e garanto que até o fim da mesma irá ficar cada vez mais claro.

---

&nbsp;

<div id='iter'/>

## 5 - Aplicando método \_\_iter__

&nbsp;

Se constatamos que nosso objeto pode receber o método \_\_iter__, então vamos aplicá-lo. Mas primeiro vamos verificar o tipo do nosso objeto.

~~~python
# Verificando o tipo do objeto conteudo
print(f'conteudo = {type(conteudo)}')
~~~

`Retorna:`

```
conteudo = <class 'str'>
```

Nosso objeto é um class 'str', então agora aplicamos a função iter( ).

~~~python
# Tornando o objeto conteudo um iterador
conteudo = iter(conteudo)
print(f'conteudo = {type(conteudo)}')
~~~

`Retorna:`

```
conteudo = <class 'str_iterator'>
```

A função iter( ) utiliza nosso método especial \_\_iter__ para tornar nosso objeto um iterador. Vamos agora novamente verificar se nosso objeto pode receber o método \_\_next__.

~~~python
# Confirmando se conteudo tem o método __next__ o tornando um iterador.
'__next__' in dir(conteudo)
~~~

`Retorna:`

```
True
```

Podemos afirmar agora que todo objeto que pode receber o método **\_\_iter__** é um **ITERÁVEL** e após recebê-lo é habilitado a receber o método **\_\_next__**, confirmando que ele também é um **ITERADOR**.

---

<div id='lazy'/>

&nbsp;

## 6 - Lógica avaliação preguiçosa (Lazy evaluation)

&nbsp;

Antes de continuarmos nosso raciocínio proponho conhecermos um pouco mais sobre a lógica avaliação preguiçosa. 

> Avaliação preguiçosa (também conhecida por avaliação atrasada) é uma técnica usada em programação para atrasar a computação até um ponto em que o resultado da computação é considerado suficiente.

Se aplicarmos o método \_\_next__ através da função next() em nosso objeto, estaremos aplicando a lógica avaliação preguiçosa, ou seja, iterando sobre nosso arquivo letra por letra.

~~~python
# iterando sobre o objeto com next().
print(next(conteudo))
print(next(conteudo))
print(next(conteudo))
print(next(conteudo))
~~~

`Retorna:`

```
T
i
p
o
```

Conseguimos fazer isso porque nosso arquivo aberto no Python é um iterável, um tipo de objeto que possibilita a ação de repetição sobre seus elementos, como listas e strings.

No Python, um objeto é considerado iterável se ele implementa o método \_\_iter__, permitindo, por exemplo, que um loop for seja executado sobre ele.

Uma solução hipotética seria ter um iterável que nos permitisse gerar uma linha por vez a cada iteração, à medida do necessário.

Esse tipo de lógica, na computação, tem nome - avaliação preguiçosa, ou, em inglês, _lazy evaluation_.

A avaliação preguiçosa, como já indica o nome, atrasa o processamento de uma expressão até que o resultado seja de fato necessário.

---

<div id='class_lazy'/>

### 6.1 - Criando uma classe aplicando Lazy evaluation

&nbsp;

A solução 1 seria criarmos uma classe que receberia nosso arquivo e aplicasse os métodos \_\_iter__ e \_\_next__, retornando, então, linha por linha do nosso arquivo. Assim, aplicando a lógica lazy evaluation, independentemente do tamanho do arquivo, nossa memória estaria armazenada somente com uma única linha por vez.  

~~~python
class File_iterator():
    """Recebe arquivo e retorna linha por linha aplicando
    lógica lazy evaluation"""

    def __init__(self, file: str):
        self.file = open(file, 'r', encoding="utf-8")
        self.linha_atual = ''

    def __iter__(self):
        """Tornando arquivo file um iterador"""
        return self

    def __next__(self):
        """Itera linha a linha do arquivo file. Retorna 
        linha do arquivo se a mesma possuir dados se não
        StopIteration"""
        self.linha_atual = self.file.readline()
        if self.linha_atual:
            return self.linha_atual
        raise StopIteration
~~~

Testando nossa base de dados com a nova class File_iterator:

~~~python
file = 'VendasTesouroDireto.csv'

file_iterator = File_iterator(file)

# Imprimindo as duas primeiras linhas
print(next(file_iterator))
print(next(file_iterator))
~~~

`Retorna:`

```
Tipo Titulo;Vencimento do Titulo;Data Venda;PU;Quantidade;Valor

Tesouro IPCA+ com Juros Semestrais;15/05/2011;22/09/2008;1678,137281;664,40;1114954,40
```

Verificando consumo em bytes de cada iteração:

~~~python
# Verificando consumo de memória em cada iteração
from sys import getsizeof

file = 'VendasTesouroDireto.csv'
file_iterator = File_iterator(file)

linha1 = next(file_iterator)
print(f'{linha1}\nLinha 1 = {getsizeof(linha1)} bytes\n')

linha2 = next(file_iterator)
print(f'{linha2}\nLinha 2 = {getsizeof(linha2)} bytes')
~~~

`Retorna:`

```
Tipo Titulo;Vencimento do Titulo;Data Venda;PU;Quantidade;Valor

Linha 1 = 113 bytes

Tesouro IPCA+ com Juros Semestrais;15/05/2011;22/09/2008;1678,137281;664,40;1114954,40

Linha 2 = 136 bytes
```

Consumimos aproximadamente 136 bytes por cada iteração, utilizando a lógica Lazy Evaluation com as aplicações dos métodos \_\_iter__ e \_\_next__ . Independentemente do tamanho do arquivo, vamos alocar sempre uma única linha na memória, tornando nossa aplicação mais robusta.

Mas não paramos por aqui. Se verificarmos a figura1, ainda temos uma forma mais elegante e "pythonica" para resolver esse problema. Vamos verificar como ficaria essa mesma solução utilizando Geradores (**yield**).

Geradores são maneiras de fazer um iterador sem toda aquela confusão de classes. Mas o estudo sobre estas foi importante  para se saber  como tudo funciona, nos mínimos detalhes.

A palavra reservada **yiled** pode ser usada no corpo de qualquer função. Isso vai transformar a sua função em um **iterável**.


---

<div id='expr_geradoras'/>

&nbsp;

## 7 - Expressões geradoras.

&nbsp;

O objetivo é criar uma função geradora para substituir nossa class File_iterator, mas primeiro proponho conhecer expressões geradoras.

Vamos fazer um exemplo da tabuada de 2 com um for, depois com uma list comprehensions e, por fim, com uma expressão geradora.

~~~python
# Criando uma lista com resultado da tabuada de 2 com um for
tabuada = []
for numero in range(1,11):
    tabuada.append(numero*2)

print(tabuada)
~~~

`Retorna:`

```
[2, 4, 6, 8, 10, 12, 14, 16, 18, 20]
```

~~~python
# Substituindo o for por uma list comprehensions
tabuada = [x*2 for x in range(1,11)]
print(tabuada)
~~~

`Retorna:`

```
[2, 4, 6, 8, 10, 12, 14, 16, 18, 20]
```

Conseguimos criar estruturas comprehensions com list [ ], set { } e dict {key, value}. Se fizermos com tuple ( ), a mesma se torna uma expressão geradora.

~~~python
# Substituindo por um tuple comprehensions(generator)
tabuada = (x*2 for x in range(1,11))
print(tabuada)
~~~

`Retorna:`

```
<generator object <genexpr> at 0x000001DBBE7067B0>
```

Retornou um objeto gerador. E se colocarmos um next( )?

~~~python
# Expressão geradora mais next( ) Aplicando Lazy Evaluetion

print(f'1º next = {next(tabuada)}')
print(f'2º next = {next(tabuada)}')
print(f'3º next = {next(tabuada)}')
~~~

`Retorna:`

```
1º next = 2
2º next = 4
3º next = 6
```

O retorno é exatamente uma lógica Lazy evaluation, ou seja, a cada next( ) é gerado um resultado da tabuada de 2. Podemos concluir que usando geradores podemos substituir nossa class File_iterator. 

---

<div id='gerador'/>

&nbsp;

## 8 - Função Geradora (Aplicando yield).

&nbsp;

Poderíamos vir diretamente para este tópico, pois vamos propor uma substituição da nossa class File_iterator por uma função geradora. Porém, julguei importante explicar sobre a possibilidade de se criar um gerador através de uma expressão geradora.

A função geradora tem a palavra mágica yield que pode nos ajudar a escrever geradores de maneira simples,  gerando  iteráveis com pouco código e sem uso dos métodos \_\_iter__ e \_\_next__. A linguagem faz isso para você. Você só se preocupa com o necessário.

~~~python
# Exemplo de função geradora
def geradora():
    yield 1
    yield 2
    yield 3

g = geradora()

print(f'{next(g)}')
print(f'{next(g)}')
print(f'{next(g)}')
~~~

`Retorna:`

```
1
2
3
```
Olha que simplicidade: a cada next( ) temos a execução de um yield, nossa função com apenas uma linha representa os métodos \_\_iter__ e \_\_next__. 

Uma função com yield é somente uma função, mas ao ser chamada se torna um gerador.

~~~python
# Função Vs Gerador
print(f'O tipo de def geradora é: {type(geradora)}\n')
print(f'O tipo de g = geradora() é: {type(g)}\n')
~~~

`Retorna:`

```
O tipo de def geradora é: <class 'function'>

O tipo de g = geradora() é: <class 'generator'>
```

Poderíamos citar inúmeros exemplos de geradores aqui, mas nosso foco é gerar uma solução para leitura de arquivo linha por linha independentemente do tamanho do mesmo.

Vamos para última etapa e a solução do problema.


---

<div id='solucao'/>

&nbsp;

## 9 - Função Geradora para ler linha a linha de um arquivo.

&nbsp;

Por fim a função geradora com apenas 3 linhas que substituirá a class seria:

~~~python
def file_iterator(file:str):
    for linha in open(file, 'r', encoding="utf-8"):
        yield linha.strip().split(';')
~~~

Solução com class:

~~~python
class File_iterator():
    """Recebe arquivo e retorna linha por linha aplicando
    lógica lazy evaluation"""

    def __init__(self, file: str):
        self.file = open(file, 'r', encoding="utf-8")
        self.linha_atual = ''

    def __iter__(self):
        """Tornando arquivo file um iterador"""
        return self

    def __next__(self):
        """Itera linha a linha do arquivo file. Retorna 
        linha do arquivo se a mesma possuir dados se não
        StopIteration"""
        self.linha_atual = self.file.readline()
        if self.linha_atual:
            return self.linha_atual
        raise StopIteration
~~~

Testando a função geradora:

~~~python
def gen(file:str):
    for linha in open(file, 'r', encoding="utf-8"):
        yield linha.strip().split(';')

file = 'VendasTesouroDireto.csv'
f = gen(file)

# Assim ele carrega linha a linha na memória e não o arquivo inteiro.
print(next(f))
print(next(f))
print(next(f))
~~~

`Retorna:`

```
['Tipo Titulo', 'Vencimento do Titulo', 'Data Venda', 'PU', 'Quantidade', 'Valor']
['Tesouro IPCA+ com Juros Semestrais', '15/05/2011', '22/09/2008', '1678,137281', '664,40', '1114954,40']
['Tesouro Prefixado com Juros Semestrais', '01/01/2010', '22/09/2008', '968,950900', '300,00', '290685,27']
```

---

&nbsp;

<div id='conclusão'/>

## Conclusão:

&nbsp;

Passamos por objetos iteráveis no python que ao receberem dados se tornam conteiners.

Passamos pelo método \_\_getitem__ para entendermos a diferença entre alocarmos um objeto inteiro na memória ou alocarmos somente parte dele com lazy evaluation.

Verificamos a diferença entre iterável (\_\_iter__) e iterador (\_\_next__).

Criamos uma class aplicando os métodos citados acima e atingimos nosso objetivo de economia de memória aplicando a lógica Lazy Evaluation.

Posteriormente, passamos por geradores e conseguimos o mesmo resultado com apenas 3 linhas de código.

Concluímos que com o uso de uma função geradora utilizando yield  conseguimos uma solução para iterar com arquivos independentemente do tamanho dos mesmos e consumimos pouca memória, tornando nossa aplicação mais robusta e pythonica com poucas linhas de código.
