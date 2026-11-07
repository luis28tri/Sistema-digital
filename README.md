# Alarma de tres ráfagas

## Estudiante

- Matrícula: 20210684
- Correo: 20210684@itla.edu.do

## Descripción

Se implementó una máquina de estados finitos (FSM) que genera tres ráfagas sonoras separadas por períodos de silencio.

La secuencia implementada es:

IDLE → ON0 → OFF0 → ON1 → OFF1 → ON2 → IDLE

## Funcionamiento

- cmd[0] actúa como señal start.
- start solamente se acepta en estado IDLE.
- Durante busy los nuevos start son ignorados.
- Cada estado ON dura 2\*K flancos habilitados.
- Cada estado OFF dura K flancos habilitados.
- Cuando en=0 todo el sistema se congela.
- speaker alterna cada dos flancos habilitados.
- Cada transición reinicia speaker y el contador de audio.

## Señales de salida

- y[0] = speaker
- y[1] = busy
- y[3:2] = índice de ráfaga
- y[15:4] = 0

## Simulación

Compilar el archivo principal:

top\_module.v

y el banco de pruebas:

tb\_student.v

en cualquier simulador compatible con Verilog 2001.

## Caso límite considerado

Si start permanece en alto cuando termina ON2 y el sistema vuelve a IDLE, la nueva secuencia se acepta en el siguiente flanco habilitado, según requiere la especificación.


