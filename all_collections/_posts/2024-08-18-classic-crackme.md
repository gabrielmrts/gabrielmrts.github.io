---
layout: post
title: Classic Crackme 0x100 - Parte 1
date: 2024-08-18
categories: ["ctf", "picoctf", "reverse engineering"]
---

Nesse post irei fazer um desafio CTF de engenharia reversa. 
Estou usando o site ```picoctf```.

Temos acesso a um binário chamado ```crackme100```, então vamos começar por ele.

Rodando um ```file crackme100``` temos:

```
crackme100: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=a064bd4ed6605f6b04abb44282ecc10fedc67684, for GNU/Linux 3.2.0, with debug_info, not stripped
```

Já temos algumas informações sobre o que é esse binário, então vou partir para o ```Ghidra```, uma ferramente open-source poderosa para fazer engenharia reversa.

![Print](/assets/imgs/1.png)

A instrução que faz esse ```if (iVar2 == 0)``` na linha 87:

![Print](/assets/imgs/2.png)

```
TEST    EAX, EAX
JNZ     LAB_00401389
```

Onde ```JNZ``` (JUMP IF NOT ZERO) da um ```JUMP``` para o label ```LAB_00401389``` que coloca a mensagem de "FAILED!" no registrador ```EDI```, e depois chama ```puts```.

Se a flag estivesse no binário (o que não está), poderíamos modificar o binário para fazer ele mostrar a flag mesmo se a senha estivesse incorreta. Simplesmente alterando a instrução ```JNZ``` para ```JZ```.

Qual a lógica nisso?

A linha ```iVar2 = memcmp(input,output,(long)(int)sVar3);``` chama o ```memcmp``` para comparar se o input é igual o output (a senha propriamente dita), ele retorna 0 caso seja igual, e valores diferentes de 0 caso seja diferente. 

O retorno do memcmp é armazenado no registrador EAX. 

Na linha ```TEST EAX, EAX``` que é equivalente a ```EAX AND EAX```, em outras palavras, se o retorno do memcmp que é armazenado em EAX, for 0 (zero), significa que o input e output são iguais, se não for 0, significa que são diferentes.

O resultado da instrução TEST é armazenada no zero flag ZF.

Nesse caso se inserirmos uma senha errada, TEST irá atualizar o ZF para 0 (por que o resultado do memcmp não resultou em 0). E isso irá fazer com que ```JNZ LAB_00401389``` seja executado.

Por outro lado, se o retorno de memcmp for 0, ZF será 1, e ```JNZ LAB_00401389``` não será executado.

Dito isso, nós podemos usar a funcionalidade ```Patch Instruction``` do Ghidra, e alterar o JNZ para JZ (Jump if Zero). Com isso, vamos ter o efeito oposto, se ZF for 0 (se errarmos a senha).

![Print](/assets/imgs/3.png)

Se gerarmos um novo binário com essa instrução alterada, podemos colocar a senha incorreta que ele nos traria a flag.

![Print](/assets/imgs/4.png)

Infelizmente nesse caso, a flag real está em um servidor remoto ao qual o picoctf disponibiliza para o desafio.

Então vamos precisar entender como a flag é feita.

No código decompilado, nós temos um array de caracteres chamado output, que possui o tamanho 51. ```char output [51];```

Mas a dor de cabeça vai ser entender o que esse for tá fazendo. Descobrindo isso, conseguimos a flag. Então vamos começar.

Continua na parte 2.
