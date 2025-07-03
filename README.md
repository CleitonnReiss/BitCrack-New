
# BitCrack

Uma ferramenta para força bruta de chaves privadas de Bitcoin. O principal objetivo deste projeto é contribuir para o esforço de resolver a [transação de quebra-cabeça do Bitcoin](https://blockchain.info/tx/08389f34c98c606322740c0be6a7125d9860bb8d5cb182c02f98461e5fa6cd15): uma transação com 32 endereços que se tornam progressivamente mais difíceis de quebrar.

# Versão corrigida para GPUs AMD e Intel
Corrigido um bug que encerrava a busca abruptamente ao utilizar a opção --keyspace com o parâmetro --rangers. O erro foi identificado e resolvido.

### Usando o BitCrack

#### Uso

Use `cuBitCrack.exe` para dispositivos CUDA e `clBitCrack.exe` para dispositivos OpenCL.

# Atualização para Windows (GPUs AMD e Intel)
Os arquivos compilados para Windows estão localizados em:
New BitCrack\x64\Release

O executável principal para Windows é:
clBitCrack.exe

# Exemplos de uso:

# Carteira 33
./clBitCrack.exe -d 1 -b 64 -t 256 -p 1024 --keyspace 100000000:1ffffffff -c 187swFMjz1G54ycVU56B7jZFHFTNVQFDiu

# Carteira 41
./clBitCrack.exe -d 1 -b 64 -t 256 -p 1024 --keyspace 10000000000:1ffffffffff -c 1L5sU9qvJeuwQUdt4y1eiLmquFxKjtHr3E

# Carteira 45
./clBitCrack.exe -d 1 -b 64 -t 256 -p 1024 --keyspace 100000000000:1fffffffffff -c 1NtiLNGegHWE3Mp9g2JPkgx6wUg4TW7bbk

# Carteira 49
./clBitCrack.exe -d 1 -b 64 -t 256 -p 1024 --keyspace 1000000000000:1ffffffffffff -c 12CiUhYVTTH33w3SPUBqcpMoqnApAV4WCF

### Nota: **clBitCrack.exe ainda é EXPERIMENTAL**, pois usuários relataram bugs críticos ao executar em alguns dispositivos AMD e Intel.

```
./clBitCrack.exe -d 1 [OPÇÕES] [ALVOS]

Onde [ALVOS] são um ou mais endereços de Bitcoin.

Opções:

-i, --in ARQUIVO
    Lê os endereços do ARQUIVO, um endereço por linha. Se ARQUIVO for "-", lê da entrada padrão (stdin)

-o, --out ARQUIVO
    Adiciona as chaves privadas ao ARQUIVO, uma por linha

-d, --device N
    Usa o dispositivo com ID igual a N

-b, --blocks BLOCOS
    Número de blocos CUDA

-t, --threads THREADS
    Threads por bloco

-p, --points NÚMERO
    Cada thread processará NÚMERO de chaves por vez

--keyspace INTERVALO
    Especifica o intervalo de chaves a ser pesquisado, onde INTERVALO está no formato:

    START:END → começa na chave START e termina na chave END  
    START:+COUNT → começa na chave START e termina em START + COUNT  
    :END → começa na chave 1 e termina na chave END  
    :+COUNT → começa na chave 1 e termina na chave 1 + COUNT

-c, --compressed  
    Procura por chaves comprimidas (padrão). Pode ser usada com -u para procurar também as não comprimidas

-u, --uncompressed  
    Procura por chaves não comprimidas. Pode ser usada com -c para procurar ambas

--compression MODO  
    Especifica o modo de compressão, onde MODO pode ser 'compressed', 'uncompressed' ou 'both'

--list-devices  
    Lista os dispositivos disponíveis

--stride NÚMERO  
    Incrementa por NÚMERO

--share M/N  
    Divide o espaço de chaves em N partes iguais e processa a parte M

--continue ARQUIVO  
    Salva/carrega o progresso a partir do ARQUIVO
```

#### Exemplos

O uso mais simples: o intervalo de chaves começará em 0 e os parâmetros CUDA serão escolhidos automaticamente:

```
./clBitCrack.exe -d 1 1FshYsUh3mqgsG29XpZ23eLjWV8Ur3VwH
```

Várias chaves podem ser procuradas ao mesmo tempo com impacto mínimo na performance. Forneça os endereços na linha de comando ou em um arquivo com um endereço por linha:

```
./clBitCrack.exe -d 1 1FshYsUh3mqgsG29XpZ23eLjWV8Ur3VwH 15JhYXn6Mx3oF4Y7PcTAv2wVVAuCFFQNiP 19EEC52krRUK1RkUAEZmQdjTyHT7Gp1TYT
```

Para iniciar a busca em uma chave privada específica, use a opção `--keyspace`:

```
./clBitCrack.exe -d 1 --keyspace 766519C977831678F0000000000 1FshYsUh3mqgsG29XpZ23eLjWV8Ur3VwH
```

A opção `--keyspace` também pode ser usada para buscar em um intervalo específico:

```
./clBitCrack.exe -d 1 --keyspace 80000000:ffffffff 1FshYsUh3mqgsG29XpZ23eLjWV8Ur3VwH
```

Para salvar o progresso periodicamente, pode-se usar a opção `--continue`. Isso é útil para retomar após uma interrupção inesperada:

```
./clBitCrack.exe -d 1 --keyspace 80000000:ffffffff 1FshYsUh3mqgsG29XpZ23eLjWV8Ur3VwH
...
GeForce GT 640   224/1024MB | 1 target 10.33 MKey/s (1,244,659,712 total) [00:01:58]
^C
./clBitCrack.exe -d 1 --keyspace 80000000:ffffffff 1FshYsUh3mqgsG29XpZ23eLjWV8Ur3VwH
...
GeForce GT 640   224/1024MB | 1 target 10.33 MKey/s (1,357,905,920 total) [00:02:12]
```

Use as opções `-b`, `-t` e `-p` para especificar o número de blocos, threads por bloco e chaves por thread:

```
./clBitCrack.exe -d 1 -b 32 -t 256 -p 16 1FshYsUh3mqgsG29XpZ23eLjWV8Ur3VwH
```

### Escolhendo os parâmetros corretos para seu dispositivo

As GPUs possuem muitos núcleos. O trabalho para os núcleos é dividido em blocos. Cada bloco contém threads.

Existem 3 parâmetros que afetam a performance: blocos, threads por bloco e chaves por thread.

- `blocks:` Deve ser múltiplo do número de unidades de computação no dispositivo. O padrão é 32.
- `threads:` Número de threads por bloco. Deve ser múltiplo de 32. O padrão é 256.
- `keys per thread:` Número de chaves que cada thread irá processar. A performance (chaves por segundo) aumenta de forma assintótica com esse valor. O padrão é 256. Aumentar esse valor fará o kernel rodar por mais tempo, mas processará mais chaves.

### Dependências de compilação

- Visual Studio 2019 (se estiver no Windows)
- Para CUDA: CUDA Toolkit 10.1
- Para OpenCL: Um SDK OpenCL (o próprio CUDA Toolkit contém um SDK OpenCL)
- Sempre use a versao 120 do OpenCl para compilar todos as arquivos ou apresentara erros no windows.

### Compilando no Windows

Abra a solução do Visual Studio.

- Compile o projeto `clKeyFinder` para uma build OpenCL.
- Compile o projeto `cuKeyFinder` para uma build CUDA.

**Nota:** Por padrão, os headers do OpenCL da NVIDIA são usados. Você pode configurar os caminhos dos headers e bibliotecas do OpenCL no arquivo de propriedades `BitCrack.props`.

### Compilando no Linux

Usando `make`:

Compilar CUDA:
```
make BUILD_CUDA=1
```

Compilar OpenCL:
```
make BUILD_OPENCL=1
```

Ou compilar ambos:
```
make BUILD_CUDA=1 BUILD_OPENCL=1
```
