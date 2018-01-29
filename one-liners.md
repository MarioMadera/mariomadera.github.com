# Scripts en una sola línea

## Monitoreo de consumo de recursos de un proceso

ver https://www.ibm.com/support/knowledgecenter/en/ssw_aix_72/com.ibm.aix.performance/mem_usage_determine_ps.htm 

```ksh
 (ps v 21758036 | head -1) && ( i=1; while [[ i -lt 10 ]]; do ps v 21758036 | tail -1; let i=i+1; sleep 60; done )
```

**Salida**

```
      PID    TTY STAT  TIME PGIN  SIZE   RSS   LIM  TSIZ   TRS %CPU %MEM COMMAND
 21758036      - A     3:36   11 230380 231500    xx  1642  1184  0.0  2.0 /softw
 21758036      - A     3:36   11 230380 231500    xx  1642  1184  0.0  2.0 /softw
 21758036      - A     3:36   11 230380 231500    xx  1642  1184  0.0  2.0 /softw

```

**Mejoras**

crear una funcion que acepte tres parametros: -p pid, -c cantidad de iteraciones, -s tiempo de espera en segundos.

