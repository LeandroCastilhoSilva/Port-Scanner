cat > README.md << 'EOF'
# Port Scanner

Um scanner de portas TCP feito em Python. Ele testa um computador (IP ou domínio) e diz quais portas estão abertas, o que provavelmente está rodando nelas, e às vezes até a versão do programa.

Fiz esse projeto pra aprender na prática como funciona um scanner de rede simples.

## Aviso importante

Só use esse programa em computadores e redes que são seus, ou que você tem permissão pra testar. Escanear portas de outras pessoas sem autorização pode ser crime, dependendo do país.

## O que ele faz

- Testa uma porta única ou um intervalo de portas (ex: 20 até 1024)
- Testa várias portas ao mesmo tempo, então é rápido
- Mostra o nome do serviço mais comum de cada porta (ex: porta 22 = SSH)
- Tenta ler a resposta do serviço, o que às vezes mostra a versão do programa rodando

## O que você precisa ter instalado

- Python 3.10 ou mais novo

Não precisa instalar nenhuma biblioteca extra, só usa o Python puro.

## Como usar

Primeiro, entra na pasta `src`:

```bash
cd src
```

### Testar uma porta só

```bash
python3 scanner.py 127.0.0.1 -p 22
```

Isso testa a porta 22 no seu próprio computador.

### Testar várias portas de uma vez

```bash
python3 scanner.py 127.0.0.1 -p 1-1000
```

Isso testa da porta 1 até a 1000.

### Testar outro computador da rede

```bash
python3 scanner.py 192.168.1.10 -p 1-1000
```

Troca `192.168.1.10` pelo IP do computador que você quer testar (só se for seu ou autorizado).

## Opções disponíveis

| Opção | O que faz | Padrão |
|---|---|---|
| `target` | IP ou nome do computador (obrigatório) | - |
| `-p` ou `--ports` | Porta única ou intervalo, ex: `22` ou `1-1000` | 22 |
| `-t` ou `--threads` | Quantas portas testar ao mesmo tempo | 50 |
| `--timeout` | Quanto tempo esperar por cada porta, em segundos | 1.0 |

Exemplo usando todas as opções:

```bash
python3 scanner.py 127.0.0.1 -p 1-500 -t 100 --timeout 0.5
```

## O que eu aprendi fazendo esse projeto

- Como funciona uma conexão TCP usando sockets em Python
- Como rodar várias tarefas ao mesmo tempo com threads, pra deixar o programa mais rápido
- Como identificar serviços rodando numa porta (banner grabbing)
- Como separar erros graves (que param o programa) de erros pequenos (que só avisam e continuam)
EOF
