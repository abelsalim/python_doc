
# Geradores, Yield e suas peculiaridades

Introduzida a partir do python 2.3, `yield` é uma palavra reservada que em
tradução livre significa produzir ou produção.

## Vantagens
Em comparação com um list comprehension, uma função geradora ou um generator
expression processa item por item ao ser requisitado, seja pelo usuário via
script, já as demais compressões tendem a armazenar o resultado da requisição em
memória através de uma variável, lista, conjunto ou dicionário.

>Com a presença de um yield em uma função, a mesma deixa de ser um  objeto
>função e torna-se um objeto gerador.

## Funcionamento  básico
Como característica principal, um gerador é capaz de _iterar_ em um _iterável_,
contudo, um gerador não tem um comportamento igual ou similar a um container -
comprehension - por exemplo.

>__Comportamento de um Comprehension__: Um comprehension tende a processar tudo
>que for requisitado, processa caso seja necessário e por fim o armazena em
>memória. Como exemplo podemos imaginar comprehension que retorne todos os
>números ímpares num intervalo de 0 a 9.
```python
In [1]: [impar for impar in range(10) if impar % 2]
Out[1]: [1, 3, 5, 7, 9]
```

>__Comportamento de um Gerador__: Um gerador recebe um iterável e processa um
>por vez, ou seja, através de um next ou um for é possível iterar em um gerador.
>Como exemplo vamos utilizar o mesmo exemplo utilizado no _comportamento de um_
>_comprehension_.
```python
In [2]: def gerador_impar():
   ...:     for impar in range(10):
   ...:         if impar % 2:
   ...:             yield impar
   ...: 

In [3]: gen = gerador_impar()

In [4]: next(gen)
Out[4]: 1

In [5]: next(gen)
Out[5]: 3

In [6]: next(gen)
Out[6]: 5

In [7]: next(gen)
Out[7]: 7

In [8]: next(gen)
Out[8]: 9
```
>Uma outra forma de regar essa mesma função é com um generator expression que
>uma sintaxe similar ao de um comprehension, mas em relação a um gerador o
>yield vem de forma implícita.
```python
In [9]: gen = (impar for impar in range(10) if impar % 2)

In [10]: next(gen)
Out[10]: 1

In [11]: next(gen)
Out[11]: 3

In [12]: next(gen)
Out[12]: 5

In [13]: next(gen)
Out[13]: 7

In [14]: next(gen)
Out[14]: 9
```

## Consumo de memória
Para isso podemos utilizar a método getsizeof da classe sys e termos uma boa
perspeciva de como as coisas funcionam. Como exemplo vamos verificar a saída de
um comprehension que gera todos os numeros ímpares num intervalo de 0 a 99999.
#### Com comprehension
```python
In [15]: from sys import getsizeof

In [16]: getsizeof([impar for impar in range(100000) if impar % 2])
Out[16]: 444376

```

#### Com geradores
```python
In [17]: from sys import getsizeof

In [18]: getsizeof(impar for impar in range(100000) if impar % 2)
Out[18]: 104
```


Ao analisarmos as duas saídas e tendo em mente que o retorno do método getsizeof
retorna o consumo em bytes, podemos ver que ao dividirmos o retorno do container
por 1024 teremos o consumo em kilobyte e se dividirmos novamente por 1024
teremos um resultado final de 0.42M de consumo, já o generator teve um consumo
de 104 bytes, ou seja uma diferença de 444.272 bytes, mas por que obtemos uma
discrepância tão grande entre as duas? Bom, como foi dito antes, o container
processa a requisição e o armazena em memória, já o gerador gera os números por
requisição, apenas retornando o próximo número sem armazenar os dados já
iterados ou os que ainda serão iterados.

>Ao ser rquisitado, um gerador _itera_ item por item aplicando um `return` no
>_iterável da vez_ através do `yield`, com isso sempre prevalecerá o consumo
>minimo de processamento e memória.



### Identificação visual de um generator através do _type object_
```python
In [19]: def func_exemplo():
    ...:     yield 1
    ...: 

In [20]: type(func_exemplo)
Out[20]: function

In [22]: type(func_exemplo())
Out[22]: generator
```


### Criando um gerador infinito com `yield`
```python
In [23]: def my_gerador():
    ...:     number = 1
    ...:     while True:
    ...:         yield number
    ...:         number += 1
    ...: 

In [24]: gerador = my_gerador()

In [25]: next(gerador)
Out[25]: 1

In [26]: next(gerador)
Out[26]: 2

In [27]: next(gerador)
Out[27]: 3
```


*Caso o iterator chegue ao fim, um 'StopIteration' será apresentado*

## Iterando com  subgeradores
Existem casos que temos que iterar em um outro gerador. Para que isso seja
realizado com eficiência e venhamos obter o resultado esperado é necessário usar
o `yield from`.

Para entendermos bem a situação vamos criar um gerador que tente iterar em outro
gerador.

___obs: a função `range` é sim um gerador, e por isso o utilizaremos.___

```python
In [28]: def my_gerador():
    ...:       yield range(100)
    ...: 

In [29]: gen = my_gerador()

In [30]: next(gen)
Out[30]: range(0, 100)
```

Como imaginado, o `yield` foi capaz apenas de retornar o próprio `range` como
sintaxe, mas não foi capaz de iterar dentro dele. Para solucionarmos isso iremos utilizar o `yield from`.
```python
In [31]: def my_gerador(ate):
   ...:     yield from range(ate)
   ...: 

In [32]: gerador = my_gerador(100)

In [33]: next(gerador)
Out[33]: 0

In [34]: next(gerador)
Out[34]: 1

In [35]: next(gerador)
Out[35]: 2
```
Agora o `yield` foi capaz de iterar corretamente em outro gerador já que está
acompanhado ao `from`




### Iterando em um gerador dentro de outro gerador com yield e for

```python
In [14]: def my_gerador(ate):
    ...:     for x in range(ate):
    ...:       yield x
    ...: 

In [15]: gerador = my_gerador(100)

In [16]: next(gerador)
Out[16]: 0

In [17]: next(gerador)
Out[17]: 1

In [18]: next(gerador)
Out[18]: 2
```


Obitivemos com esse exemplo o retorno uma impressão do próprio objeto generator
sem ocorrer a iteração.

## Utilizando Expressões geradores
Expressões geradoras nada mais é do que geradores em container.
```python
# Generator expression que retorna os números ímpares de 0 a 10
In [22]: gen = (x for x in range(10) if x % 2)

In [23]: next(gen)
Out[23]: 1

In [24]: next(gen)
Out[24]: 3

In [25]: next(gen)
Out[25]: 5

In [26]: next(gen)
Out[26]: 7

In [27]: next(gen)
Out[27]: 9
```
Em expressões geradoras temos um `yield` oculto que transforma o catainer em
definitivamente um gerador, e com isso podemos itera-lo com o método next. 
