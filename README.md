# TRABALHO 1 — DIAGNÓSTICO DE PROCESSOS EM LINUX

## Sistemas Operacionais — 2 ADS A 2026/2

**Alunos**: Brenno Patrick Galvão Pereira / Dmylle Charis de Andrade Santos
**Aplicação/processo analisado:** Mozilla Firefox  
**Sistema operacional:** Ubuntu Linux — versão: 26.04.01
**Ambiente:** máquina física  
**Data:**  

## 1. Aplicação escolhida

A aplicação escolhida foi o Mozilla Firefox. Escolhemos o Firefox porque ele é um processo de usuário que pode ser monitorado, pausado, retomado e encerrado sem prejudicar o funcionamento do sistema.

Na coleta principal, o processo analisado tinha PID **8976**. Para complementar a evidência dos sinais, o Firefox foi aberto novamente e recebeu o PID **14693**. Por isso aparecem dois PIDs nas evidências.

## 2. Como a aplicação foi executada

O Firefox foi iniciado normalmente pela interface gráfica do Ubuntu. O processo foi localizado pelo terminal com:

```bash
pgrep firefox
```

Na coleta principal, o resultado foi `8976`.

## 3. PID e PPID

Foi utilizado:

```bash
ps -o pid,ppid,stat,pri,ni,%cpu,%mem,cmd -p 8976
```

Uma das coletas mostrou:

```text
PID   PPID STAT PRI NI %CPU %MEM CMD
8976  2174 Sl    19  0  2.4  2.5 /snap/firefox/8763/usr/lib/firefox/firefox
```

O PID identifica o processo Firefox. O PPID identifica o processo pai atual. Para verificar o PID 2174:

```bash
ps -p 2174 -o pid,ppid,stat,cmd
```

Resultado:

```text
2174  1706  Rsl  /usr/bin/gnome-shell --mode=ubuntu
```

Assim, no momento da coleta, o processo pai atual do Firefox era o GNOME Shell.

Evidência: `evidencias/01-pid-ppid/pid-ppid.txt`

## 4. Árvore de processos

O comando `pstree -p` mostrou a relação:

```text
systemd(1)
└── systemd(1706)
    └── gnome-shell(2174)
        └── firefox(8976)
```

Também foram observados componentes/processos relacionados ao Firefox, como `forkserver`, `RDD Process`, `Socket Process`, `Utility Process`, `Web Content` e `WebExtensions`.

Com isso, foi possível observar como os processos ficam organizados e que o Firefox utiliza vários componentes para realizar suas tarefas.

Evidência: `evidencias/02-arvore-processos/arvore-processos.txt`

## 5. Estado, CPU e memória

O campo `STAT` apareceu inicialmente como `Sl`. O `S` indica estado de espera (sleeping) e o `l` indica que o processo possui múltiplas threads.

Foram feitas medições em momentos diferentes:

| Momento | CPU | Memória |
|---|---:|---:|
| 1 | 2,4% | 2,5% |
| 2 | 2,0% | 3,6% |
| Outra coleta registrada | 3,9% | 6,7% |

Os valores mudaram entre as coletas porque o consumo do Firefox não é sempre igual. Ele depende do que estava sendo feito no navegador naquele momento.

Evidência: `evidencias/03-estado-recursos/estado-recursos.txt`

## 6. Prioridade e nice

Inicialmente:

```text
PID   PRI NI
8976  19  0
```

Foi executado:

```bash
renice 5 -p 8976
```

Resultado:

```text
8976 (process ID) old priority 0, new priority 5
```

Depois:

```text
PID   PRI NI
8976  14  5
```

O nice passou de 0 para 5. Com um nice maior, o processo passa a ter menor prioridade relativa na disputa por CPU.

A evidência dessa parte está junto de `evidencias/03-estado-recursos/estado-recursos.txt`.

## 7. Threads

Foi utilizado:

```bash
ps -L -p 8976
```

A consulta mostrou várias threads do Firefox. Entre os nomes observados estavam `Socket Thread`, `HTML5 Parser`, `JS Watchdog`, `Renderer`, `Compositor`, `ImageIO` e outras.

O arquivo `/proc/8976/status` registrou:

```text
Threads: 83
```

Um processo representa a aplicação em execução. As threads são unidades de execução dentro desse processo e compartilham recursos dele.

Evidência: `evidencias/04-threads/threads.txt`

## 8. Informações em /proc

Foram analisadas quatro fontes:

- `/proc/8976/status`: estado, PPID, memória e quantidade de threads.
- `/proc/8976/limits`: limites de recursos do processo.
- `/proc/8976/cmdline`: comando/caminho usado para executar o Firefox.
- `/proc/8976/io`: dados de entrada e saída.

Alguns resultados importantes foram `VmRSS: 151416 kB`, `Threads: 83`, limite máximo de arquivos abertos de `524288` e caminho `/snap/firefox/8763/usr/lib/firefox/firefox`.

Evidência: `evidencias/05-proc/proc.txt`

## 9. Sinais

Na primeira execução foram feitos testes com SIGSTOP, SIGCONT e SIGTERM. Como não havia sido feita uma consulta do estado entre STOP e CONT, foi realizada uma coleta complementar com o Firefox aberto novamente. O novo PID foi **14693**.

Antes do STOP:

```text
PID    PPID STAT PRI NI %CPU %MEM
14693  2174 Sl    19  0  27.8  5.1
```

Depois:

```bash
kill -STOP 14693
```

A nova consulta mostrou:

```text
14693  2174 Tl    19  0  13.7  3.7
```

A letra `T` mostrou que o Firefox estava suspenso.

Em seguida:

```bash
kill -CONT 14693
```

Resultado:

```text
14693  2174 Rl    19  0  11.5  3.7
```

O processo deixou o estado `T` e voltou à execução.

Por último:

```bash
kill -TERM 14693
sleep 2
ps -p 14693
```

O `ps` exibiu somente o cabeçalho, sem o PID 14693, comprovando que o processo havia sido encerrado.

Evidência: `evidencias/06-sinais/sinais.txt`

## 10. Conclusão

O experimento permitiu acompanhar o Firefox como um processo real no Ubuntu. Foi possível identificar PID e PPID, observar a árvore de processos, acompanhar estado e uso de recursos, verificar threads, consultar informações em `/proc` e modificar o valor de nice.

Os sinais também mostraram de forma prática o controle do Linux sobre os processos. O SIGSTOP suspendeu o Firefox, o SIGCONT permitiu a retomada e o SIGTERM encerrou o processo de forma controlada.

Com o experimento, conseguimos entender melhor na prática os conceitos de gerenciamento de processos vistos na disciplina.
