# Comandos utilizados

Os comandos abaixo estão organizados de acordo com o experimento.

```bash
# Localização do Firefox
pgrep firefox

# PID, PPID, estado, prioridade, nice, CPU e memória
ps -o pid,ppid,stat,pri,ni,%cpu,%mem,cmd -p 8976

# Processo pai
ps -p 2174 -o pid,ppid,stat,cmd

# Árvore de processos
pstree -p 8976
pstree -p

# Novas consultas de CPU e memória
ps -o pid,ppid,stat,pri,ni,%cpu,%mem,cmd -p 8976

# Threads
ps -L -p 8976

# /proc
cat /proc/8976/status
cat /proc/8976/limits
cat /proc/8976/cmdline
cat /proc/8976/io

# Prioridade
renice 5 -p 8976
ps -o pid,pri,ni,cmd -p 8976

# Sinais da coleta principal
kill -STOP 8976
kill -CONT 8976
kill -TERM 8976
sleep 2
ps -p 8976

# Coleta complementar dos sinais
pgrep firefox
ps -o pid,ppid,stat,pri,ni,%cpu,%mem,cmd -p 14693
kill -STOP 14693
ps -o pid,ppid,stat,pri,ni,%cpu,%mem,cmd -p 14693
kill -CONT 14693
ps -o pid,ppid,stat,pri,ni,%cpu,%mem,cmd -p 14693
kill -TERM 14693
sleep 2
ps -p 14693
```

Durante o experimento também ocorreram alguns erros de digitação em comandos `ps`, como `%men` no lugar de `%mem`. O comando foi corrigido e executado novamente. Os resultados usados como evidência são os das execuções corretas.
