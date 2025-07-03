
# BitCrack

Uma ferramenta para força bruta de chaves privadas de Bitcoin. O principal objetivo deste projeto é contribuir para o esforço de resolver a [transação de quebra-cabeça do Bitcoin](https://blockchain.info/tx/08389f34c98c606322740c0be6a7125d9860bb8d5cb182c02f98461e5fa6cd15): uma transação com 32 endereços que se tornam progressivamente mais difíceis de quebrar.

### Usando o BitCrack

#### Uso

Use `cuBitCrack.exe` para dispositivos CUDA e `clBitCrack.exe` para dispositivos OpenCL.

### Nota: **clBitCrack.exe ainda é EXPERIMENTAL**, pois usuários relataram bugs críticos ao executar em alguns dispositivos AMD e Intel.

**Nota para usuários Intel:**

Há um bug na implementação do OpenCL da Intel que afeta o BitCrack. Detalhes aqui: https://github.com/brichard19/BitCrack/issues/123

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

### Apoie este projeto

Se você achou este projeto útil e gostaria de apoiá-lo, considere fazer uma doação. Seu apoio é muito apreciado!

**BTC**: `1LqJ9cHPKxPXDRia4tteTJdLXnisnfHsof`  
**LTC**: `LfwqkJY7YDYQWqgR26cg2T1F38YyojD67J`  
**ETH**: `0xd28082CD48E1B279425346E8f6C651C45A9023c5`

### Contato

Envie dúvidas ou comentários para:  
📧 **bitcrack.project@gmail.com**
