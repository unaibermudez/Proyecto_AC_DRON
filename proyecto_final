;#include "80c552.h"	; añadir la libreria 80c552.hç

;#################################################################################################################################
;                                                      ETIQUETAS
;#################################################################################################################################
;----variables----
ESTADO        EQU    R0
EVENTO        EQU    R1
BATERIA       EQU    R2


;----puertos----
desp_ater     EQU    P0.7
par_avan      EQU    P0.6
auto_man      EQU    P0.5
g_izda        EQU    P0.3
g_dcha        EQU    P0.2
sube          EQU    P0.1
baja          EQU    P0.0
LED_rojo      EQU    P1.4
LED_verde     EQU    P1.5
DISPLAY       EQU    P2

;----variables----
tick_10ms     EQU    0X20.0
tick_500ms    EQU    0X20.1
tick_8s       EQU    0X20.2
tick_10s      EQU    0X20.3
tick_25s      EQU    0X20.4
tick_ADC_alt  EQU    0x20.5
tick_ADC_frnt EQU    0x20.6
tick_ADC_bat  EQU    0x20.7
tick_automan  EQU    0x21.1
bateria_baja  EQU    0x21.2
bit_aux_avanza EQU   0x21.3  




est_ahora            EQU    0x22
est_antes            EQU    0x23
est_cambio           EQU    0x24   
est_cambio_sube      EQU    0x25
est_cambio_baja      EQU    0x26
est_aux              EQU    0x27


cont_10ms     EQU    0x30
cont_500ms_8s EQU    0x31
cont_500ms_10s EQU   0x32
cont_500ms_25s EQU   0x33

boton_antes   EQU    0x34
boton_ahora   EQU    0x35
boton_sube    EQU    0x28.0
boton_baja    EQU    0x28.1 






;----PWM----
PWMP          EQU    0xFE
PWM0          EQU    0xFC
PWM1          EQU    0xFD

;----ADC----
ADCON         EQU    0xC5
ADCH          EQU    0xC6

;----TIMER----
;TCON          EQU    0x88
;TMOD          EQU    0x89


;*********************************************************************************************************************************


ORG 	0x00
	AJMP	INICIO		
	

;#################################################################################################################################
;                                                      INTERRUPCIONES
;#################################################################################################################################

ORG    0x0B                ;INTERRUPCION TIMER0
	ACALL SUB_TIMER     ; 10ms
	RETI



ORG    0x53                ;INTERRUPCION POR FIN DE CONVERSION ADC
	PUSH  PSW
	PUSH  ACC
	ACALL SUB_ADC   
	POP   ACC
	POP   PSW
	RETI



;#################################################################################################################################
;                                                      PROGRAMA PRINCIPAL
;#################################################################################################################################

ORG 	0x80

INICIO:	
	ACALL	INICIALIZAR      ;Se llama a las inicializaciones de programa 
BUCLE:
       ACALL  GEN_EVE          ;Se llama al generador de eventos     
	ACALL	MAQ_ESTADOS      ;Se llama al la máquina de estados
	AJMP 	BUCLE


;#################################################################################################################################
;                                                      INICIALIZACIONES
;#################################################################################################################################

INICIALIZAR:
       MOV 	ESTADO,#0X00         ; Inicializamos variable Estado a 0
	MOV 	EVENTO,#0X00         ; Inicializamos variable Evento a 0
       MOV 	BATERIA,#0X00        ; Inicializamos variable Bateria a 0 hasta que se lea
       MOV    PWM0, #0xFF          ; PWM0 %0= parado
       MOV    PWM1, #0xFF          ; PWM1 %0= parado
       MOV    PWMP, #0x5D
       CLR    P1.6
       CLR    P1.7
       CLR    par_avan      
       CLR    auto_man      
       CLR    g_izda        
       CLR    g_dcha        
       CLR    sube          
       CLR    baja          
       CLR    LED_rojo
       CLR    LED_verde
       CLR    desp_ater
       ANL	ADCON, #11011111b    ;Poner el modo ADC: SOFTWARE ADEX
       CLR    tick_10ms    
       CLR    tick_500ms    
       CLR    tick_8s      
       CLR    tick_10s      
       CLR    tick_25s     
       CLR    tick_ADC_alt  
       CLR    tick_ADC_frnt 
       CLR    tick_ADC_bat  
       CLR    tick_automan 
       CLR    bateria_baja
       MOV    boton_antes,#0    
       MOV    est_antes,#0            
       MOV    est_cambio,#0              
       MOV    est_cambio_sube,#0      
       MOV    est_cambio_baja,#0      
       MOV    est_aux,#0              
       SETB   IE.5
       SETB   IE.6          ; habilita interrupciones por adc
       SETB   IE.7          ; EA habilita todas las interrupciones
       ACALL  PROGRAM_TIMER
       RET


;#################################################################################################################################
;                                                      GENERADOR DE EVENTOS
;#################################################################################################################################

GEN_EVE:
       MOV    boton_ahora, P0           ;guardamos el estado del puerto 0
       ANL    boton_ahora, #10000000b    ; solo nos interesa el bit 7
       MOV    A,boton_ahora
       XRL    A,boton_antes             ;XOR con antes para ver si hay cambio 
       JNZ    CAMBIO_EN_BOTON           ;salto si acc!=0 , si hay cambio  
       JB     tick_ADC_bat, EV_7
       JB     tick_ADC_alt, EV_8
       JB     tick_ADC_frnt, EV_9
       JB     tick_automan, EV_10
       JB     tick_25s, EV_6   
       JB     tick_10s, EV_5
       JB     tick_8s, EV_4
       JB     tick_500ms, EV_3
       JB     tick_10ms, EV_2   
       MOV    EVENTO,#0X00
       RET
       

CAMBIO_EN_BOTON:
       ;hay un cambio de flanco en el boton
       MOV    est_cambio,A  
       ANL    A,boton_ahora        ;AND est_cambio con boton_ahora para ver si es ascendente
       JNZ     ENCIENDE
       ;apaga
       MOV    boton_antes,boton_ahora
       MOV    EVENTO,#0X02  ;apaga
       RET

ENCIENDE:
       MOV    boton_antes,boton_ahora
       MOV    EVENTO,#0X01  ;enciende
       RET
       

EV_2:
       CLR    tick_10ms
       MOV 	EVENTO,#0X03
       RET

EV_3:
       CLR    tick_500ms
       MOV 	EVENTO,#0X04
       RET

EV_4:
       CLR    tick_8s
       MOV 	EVENTO,#0X05
       RET  

EV_5:
       CLR    tick_10s
       MOV 	EVENTO,#0X06
       RET

EV_6:
       CLR    tick_25s
       MOV 	EVENTO,#0X07
       RET

EV_7:
       CLR    tick_ADC_bat
       MOV 	EVENTO,#0X08
       RET

EV_8:
       CLR    tick_ADC_alt
       MOV 	EVENTO,#0X09
       RET

EV_9:
       CLR    tick_ADC_frnt
       MOV 	EVENTO,#0X0A
       RET

EV_10:
       CLR    tick_automan
       MOV 	EVENTO,#0X0B
       RET



;#################################################################################################################################
;                                                      MAQUINA DE ESTADOS
;#################################################################################################################################

MAQ_ESTADOS:
	MOV	A,ESTADO		;Se mueve el valor del Estado Actual al Acumulador
	RL 	A			;Rotación a la izqu del valor del Acc (multiplicación por 2)
	MOV	DPTR,#LISTA_EST    	;Mover el valor del dato al DataPointer
	JMP	@A+DPTR			;Se salta al estado correspondiente de la tabla
       
LISTA_EST:
	AJMP	REPOSO			;ESTADO 0
	AJMP 	DESPEGUE		;ESTADO 1
	AJMP 	VUELO_ESTABLE		;ESTADO 2
	AJMP	MANUAL			;ESTADO 3
	AJMP 	ATERRIZAJE		;ESTADO 4
       AJMP   SUBIR    		;ESTADO 5
      	AJMP 	BAJAR    		;ESTADO 6
	AJMP 	GIRO_DCHA		;ESTADO 7
       AJMP   GIRO_IZQ             ;ESTADO 8






;#################################################################################################################################
;                                                      Estado 0: REPOSO
;#################################################################################################################################

REPOSO:	
	ACALL 	MAQ_EVENTOS_0		;Se llama a la máquina de eventos del Estado 0
	RET

MAQ_EVENTOS_0:
	MOV	A,EVENTO		;Trasladamos el valor del Evento actual al Acc
	RL	A			;Rotación a la izqu del valor del Acc (multiplicación por 2)
	MOV	DPTR,#LISTA_EV_0	;Mueve el dato al data pointer y nos dirige en la lista de eventos a donde queremos ir
	JMP	@A+DPTR		;Salto indirecto a la dir del dptr+Acc para colocarnos en el lugar de la lista correcto

LISTA_EV_0:
       RET                         
       NOP                  ;EVENTO 0
       AJMP	EVENTOS0_1	;EVENTO 1 encendido
       RET                         
       NOP                  ;EVENTO 2
	AJMP	EVENTOS0_3	;EVENTO 3 tick_10ms
       RET                         
	NOP                  ;EVENTO 4
       RET                         
	NOP                  ;EVENTO 5 
       AJMP   EVENTOS0_6    ;EVENTO 6 tick_10s
       RET
       NOP                  ;EVENTO 7 
       AJMP	EVENTOS0_8    ;EVENTO 8 tick_ADC_bat
       AJMP   EVENTOS0_9    ;EVENTO 9 tick_ADC_alt 
       RET                         
	NOP                  ;EVENTO 10
       RET                         
	NOP                  ;EVENTO 11
       RET

EVENTOS0_1:
       ;encendido
       MOV    PWM0,#0x4D    ; PWM0 AL 70% (SUBIR)
	MOV    ESTADO,#0x01  ; CAMBIAR A ESTADO 1: DESPEGUE
       RET

EVENTOS0_3:
       ;tick_10ms
       ;iniciar conversion altura
       CLR    tick_10ms
       ANL    ADCON,#11111000b ; canal P5.0 (altura)    
       ORL 	ADCON,#08h	   ;Enciende el ADC 
       RET


EVENTOS0_6:
       ;tick_10s
       ;iniciar conversion bateria
       CLR    tick_10s
       ANL    ADCON,#11111000b
	ORL    ADCON,#0x02      ; canal P5.2 (bateria) 
       ORL 	ADCON,#08h	   ;Enciende el ADC 
       RET

EVENTOS0_8:
       ;tick_ADC_bat
       CLR    tick_ADC_bat
	ACALL  MOSTRAR_DISPLAY  ; mostrar valor en el display
       RET

EVENTOS0_9:
       ;tick_ADC_alt
       CLR   tick_ADC_alt
	MOV   A,ADCH
	JNZ   EVENTOS0_9_FIN     ; salta si ADCH distinto de 0
	MOV   PWM0,#0xFF         ; PWM0 0% (parado)
	MOV   PWM1,#0xFF         ; PWM1 0% (parado)
	CLR   P1.7
	CLR   P1.6

EVENTOS0_9_FIN:
	RET
;#################################################################################################################################
;                                                      Estado 1: DESPEGUE
;#################################################################################################################################


DESPEGUE:	
	ACALL 	MAQ_EVENTOS_1		;Se llama a la máquina de eventos del Estado 1
	RET

MAQ_EVENTOS_1:
	MOV	A,EVENTO		;Trasladamos el valor del Evento actual al Acc
	RL	A			;Rotación a la izqu del valor del Acc (multiplicación por 2)
	MOV	DPTR,#LISTA_EV_1	;Mueve el dato al data pointer y nos dirige en la lista de eventos a donde queremos ir
	JMP	@A+DPTR			;Salto indirecto a la dir del dptr+Acc para colocarnos en el lugar de la lista correcto

LISTA_EV_1:
       RET                         
       NOP                  ;EVENTO 0
       RET                         
       NOP                  ;EVENTO 1
       AJMP	EVENTOS1_2	;EVENTO 2 apagado
	AJMP	EVENTOS1_3	;EVENTO 3 tick_10ms
       RET                        
	NOP                  ;EVENTO 4
       RET                        
	NOP                  ;EVENTO 5 
       AJMP   EVENTOS1_6    ;EVENTO 6 tick_10s
       RET
       NOP                  ;EVENTO 7 
       AJMP	EVENTOS1_8    ;EVENTO 8 tick_ADC_bat
       AJMP   EVENTOS1_9    ;EVENTO 9 tick_ADC_alt 
       RET                         
	NOP                  ;EVENTO 10
       AJMP   EVENTOS1_11   ;EVENTO 11 tick_automan  
       RET

EVENTOS1_2:
       ;apagado
       MOV    PWM0, #0xB3     ; PWM0 %30= bajar
      	MOV    PWM1, #0xFF     ; PWM1 %0= ni gira ni avanza
       MOV    ESTADO, #0x04   ; cambiar a estado ATERRIZAJE
       RET

EVENTOS1_3:
       ; tick_10ms
       ; iniciar conversion altura
       CLR    tick_10ms
	ANL    ADCON,#11111000b      ; canal P5.0 (altura) y empieza a combertir
       ORL 	ADCON,#08h	   ;Enciende el ADC 
       RET


EVENTOS1_6:
       ; tick_10s
       ;iniciar conversion bateria
       CLR    tick_10s
       ANL    ADCON,#11111000b
	ORL    ADCON,#0x02      ; canal P5.2 (bateria) y empieza conversion
       ORL 	ADCON,#08h	   ;Enciende el ADC 
       RET

EVENTOS1_8:
       ;tick_ADC_bat
       CLR    tick_ADC_bat
	ACALL  MOSTRAR_DISPLAY ; mostrar valor en el display
       JB     bateria_baja,EVENTOS1_8_CONTINUA
       RET

EVENTOS1_8_CONTINUA:
       CLR    bateria_baja
       MOV    PWM0, #0xB3     ; PWM0 %30= bajar
      	MOV    PWM1, #0xFF     ; PWM1 %0= ni gira ni avanza
       MOV    ESTADO, #0x04   ; cambiar a estado ATERRIZAJE
       RET

EVENTOS1_9:
       ;tick_ADC_alt
       CLR    tick_ADC_alt
	MOV    A,ADCH         ; mover el valor convertido al acumulador
	SUBB   A,#0x55             ;; comparar con 1m
	JNC     EVENTOS1_9_CONTINUA ; salta si carry=0 que significa que mayor o igual que 100cm
	RET
EVENTOS1_9_CONTINUA:
	SETB   tick_automan   ; ha llegado a 1 metro
	MOV    PWM0,#0x80     ; PWM0 al %50 mantiene altura
       RET

EVENTOS1_11:
       ;tick_automan 
       CLR   tick_automan
	JB    auto_man,MODO_MANUAL
	MOV   PWM1,#0x80         ; PWM1 %50 activar
	SETB  P1.7
	SETB  P1.6               ; PWM1 AVANZA
       ACALL PROGRAM_TIMER
	MOV   ESTADO,#0x02       ; cambiar a VUELO ESTABLE
	SETB  DISPLAY.7
	RET

MODO_MANUAL:
	CLR   DISPLAY.7
	MOV   ESTADO,#0x03         ; cambiar a MANUAL
       RET


;#################################################################################################################################
;                                                      Estado 2: VUELO ESTABLE
;#################################################################################################################################


VUELO_ESTABLE:	
	ACALL 	MAQ_EVENTOS_2       	;Se llama a la máquina de eventos del Estado 2
	RET

MAQ_EVENTOS_2:
	MOV	A,EVENTO		;Trasladamos el valor del Evento actual al Acc
	RL	A			;Rotación a la izqu del valor del Acc (multiplicación por 2)
	MOV	DPTR,#LISTA_EV_2	;Mueve el dato al data pointer y nos dirige en la lista de eventos a donde queremos ir
	JMP	@A+DPTR		;Salto indirecto a la dir del dptr+Acc para colocarnos en el lugar de la lista correcto

LISTA_EV_2:
       RET                         
       NOP                  ;EVENTO 0
       RET                         
       NOP                  ;EVENTO 1
       AJMP	EVENTOS2_2	;EVENTO 2 apagado
	AJMP	EVENTOS2_3	;EVENTO 3 tick_10ms
       RET                        
	NOP                  ;EVENTO 4 
       RET                        
	NOP                  ;EVENTO 5  
       AJMP   EVENTOS2_6    ;EVENTO 6  tick_10s
       AJMP   EVENTOS2_7    ;EVENTO 7  tick_25s
       AJMP	EVENTOS2_8    ;EVENTO 8  tick_ADC_bat
       AJMP   EVENTOS2_9    ;EVENTO 9  tick_ADC_alt 
       AJMP   EVENTOS2_10   ;EVENTO 10 tick_ADC_frnt
       RET                        
	NOP                  ;EVENTO 11
       RET



EVENTOS2_2:
       ;apagado
       MOV    PWM0, #0xB3     ; PWM0 %30= bajar
      	MOV    PWM1, #0xFF     ; PWM1 %0= ni gira ni avanza
       MOV    ESTADO, #0x04   ; cambiar a estado ATERRIZAJE

       RET
EVENTOS2_3:
       ;tick_10ms
       ;iniciar conversion altura
       CLR    tick_10ms
       ANL    ADCON,#11111000b ; canal P5.0 (altura) 
       ORL 	ADCON,#08h	   ;Enciende el ADC 
       RET


EVENTOS2_6:
       ;tick_10s
       ;iniciar conversion bateria
       CLR    tick_10s
       ANL    ADCON,#11111000b
	ORL    ADCON,#0x02      ; canal P5.2 (bateria) 
       ORL 	ADCON,#08h	   ; Enciende el ADC 
       RET

EVENTOS2_7:
       ;tick_25s
       CLR    tick_25s
       CLR    P1.7             ; se pone a girar a la izq
       MOV    ESTADO,#0x08     ; cambiar a GIRO IZQ
       RET

EVENTOS2_8:
       ;tick_ADC_bat
       CLR    tick_ADC_bat
	ACALL  MOSTRAR_DISPLAY  ; mostrar valor en el display
       JB     bateria_baja,EVENTOS2_8_CONTINUA
       RET

EVENTOS2_8_CONTINUA:
       CLR    bateria_baja
       MOV    PWM0, #0xB3     ; PWM0 %30= bajar
      	MOV    PWM1, #0xFF     ; PWM1 %0= ni gira ni avanza
       MOV    ESTADO, #0x04   ; cambiar a estado ATERRIZAJE
       RET

EVENTOS2_9:
       ;tick_ADC_alt
	CLR   tick_ADC_alt
       ANL   ADCON, #11111000b  
	ORL   ADCON,#0x01         ; canal P5.1 (frente)
       ORL   ADCON,#08h	     ; Enciende el ADC 
	MOV   A,ADCH  
	SUBB  A,#0x44             ; 0.8m
	JNC   EVENTOS2_9_CONTINUA_1 ;salta si carry=0 que significa que mayor o igual que 80cm
	MOV   PWM0,#0x4D          ; PWM0 al 70% sube
	CLR   P1.7
	CLR   P1.6                ; PWM1 parado
	MOV   ESTADO,#0x05        ; cambia a estado SUBE
	RET

EVENTOS2_9_CONTINUA_1:
	MOV   A,ADCH
	SUBB  A,#0x66              ; 1.2m
	JC    EVENTOS2_9_CONTINUA_2; salta si carry=1 que significa que menor que 120cm
	MOV   PWM0,#0x4D           ; PWM0 al 30% baja
	CLR   P1.7
	CLR   P1.6                 ; PWM1 parado
	MOV   ESTADO,#0x06         ; cambia a estado BAJA
	RET
EVENTOS2_9_CONTINUA_2:
	RET

EVENTOS2_10:
       ;tick_ADC_frnt
       CLR   tick_ADC_frnt
	MOV   A,ADCH
	SUBB  A,#0x22              ;40cm
	JNC   EVENTOS2_10_CONTINUA  ;salta si carry=0 que significa que mayor o igual que 40cm
	SETB  P1.7
	CLR   P1.6                 ; PWM1 giro derecha
       ACALL PROGRAM_TIMER        ; reinicia el timer
	MOV   ESTADO,#0x07         ; cambia a estado GIRO_DCHA
       RET

EVENTOS2_10_CONTINUA:
       RET

;#################################################################################################################################
;                                                      Estado 3: MANUAL
;#################################################################################################################################


MANUAL:	
	ACALL 	MAQ_EVENTOS_3       	;Se llama a la máquina de eventos del Estado 3
	RET

MAQ_EVENTOS_3:
	MOV	A,EVENTO		;Trasladamos el valor del Evento actual al Acc
	RL	A			;Rotación a la izqu del valor del Acc (multiplicación por 2)
	MOV	DPTR,#LISTA_EV_3	;Mueve el dato al data pointer y nos dirige en la lista de eventos a donde queremos ir
	JMP	@A+DPTR		;Salto indirecto a la dir del dptr+Acc para colocarnos en el lugar de la lista correcto

LISTA_EV_3:
       RET                         
       NOP                  ;EVENTO 0
       RET                         
       NOP                  ;EVENTO 1
       AJMP	EVENTOS3_2	;EVENTO 2 apagado
	AJMP	EVENTOS3_3	;EVENTO 3 tick_10ms
       RET                        
	NOP                  ;EVENTO 4
       RET                        
	NOP                  ;EVENTO 5 
       AJMP   EVENTOS3_6    ;EVENTO 6 tick_10s
       RET                        
	NOP                  ;EVENTO 7
       AJMP	EVENTOS3_8    ;EVENTO 8  tick_ADC_bat
       RET                        
	NOP                  ;EVENTO 9
       RET                        
	NOP                  ;EVENTO 10
       RET                        
	NOP                  ;EVENTO 11
       RET



EVENTOS3_2:
       ;apagado
       MOV    PWM0, #0xB3     ; PWM0 %30= bajar
      	MOV    PWM1, #0xFF     ; PWM1 %0= ni gira ni avanza
       MOV    ESTADO, #0x04   ; cambiar a estado ATERRIZAJE
       RET

EVENTOS3_3: 
       ;tick_10ms
       ;comprobar estado botones y tablas pwm
       CLR     tick_10ms
       MOV     est_ahora,P0          ; guardar estado actual del puerto  
       MOV     A, est_ahora
       ANL     A,#01001111b         ; solo nos interesan los bits 0,1,2,3,6 (baja, sube, derecha, izquierda, par_avan)
       MOV     est_ahora,A 
       MOV     A,est_ahora           ; estado actual al acumulador
       XRL     A,est_antes           ; XOR estado actual con el acumulador pondra a 1 los bits que han cambiado
       JZ      EVENTOS3_3_SIN_CAMBIO        ; si el acc es 0 no ha cambiado nada, salimos
       MOV     est_cambio,A             ; guardar en est_cambio los bits que han cambiado
       ANL     A,est_ahora           ; AND est_ahora con est_cambio para saber que bits son ascendentes
       MOV     est_cambio_sube,A        ; guardar bits ascendentes en est_cambio_sube
       MOV     A,est_cambio             ; est_cambio al acumulador
       ANL     A,est_antes           ; AND est_antes con est_cambio para saber que bits son descendentes
       MOV     est_cambio_baja,A      ; guardar bits descendentes en est_cambio_baja
       ; Paro/Avanza ;
       JNB     est_cambio_sube.6, EVENTOS3_3_CONTINUA  ;salta si no ha habido un bit ascendente en el bit 6 (paro/avanza)
       SETB    est_antes.6         ;el bit 6 (paro/avanza) a 1
       SETB    bit_aux_avanza      ;ha habido un cambio ascendente en el bit de avanza
       AJMP    EVENTOS3_3_FIN


EVENTOS3_3_SIN_CAMBIO:
       MOV     est_antes,est_ahora  ; guardar estado actual en est_antes
       RET    


EVENTOS3_3_CONTINUA:
       JNB     est_cambio_baja.6,EVENTOS3_3_FIN     ;salta si no ha habido un bit descendente en el bit 6 (paro/avanza)
       CLR     bit_aux_avanza      ;ha habido un cambio descendente en el bit de avanza
       AJMP    EVENTOS3_3_FIN


EVENTOS3_3_FIN:
       MOV     est_antes,est_ahora  ; guardar estado actual en est_antes
       JB      bit_aux_avanza, EVENTOS3_3_AUX
       CLR     est_aux.4
       CLR     est_aux.6                      ;el bit 6 de est_aux a 0
       CLR     est_aux.7                      ;el bit 7 de est_aux a 0
       CLR     est_aux.5                      ;el bit 5 de est_aux a 0
       ANL     est_cambio_sube,#00001111b     ;solo nos interesan los bits 0,1,2,3
       ANL     est_cambio_baja,#00001111b     ;solo nos interesan los bits 0,1,2,3
       MOV     A, est_aux
       XRL     A,est_cambio_sube        ;OR est_aux con est_cambio_sube para saber que bits son ascendentes
       XRL     A,est_cambio_baja        ;XOR est_aux con est_cambio_baja para saber que bits son descendentes
       MOV     est_aux,A
       ACALL   TABLA_PWM0              ;llama a la tabla de pwm0
       MOV     PWM0,A
       ACALL   TABLA1_PWM1              ;llama a la tabla1 de pwm1
       MOV     PWM1,A
       ACALL   TABLA2_PWM1
       ACALL   PWM1_AUX
       RET

EVENTOS3_3_AUX:
       SETB    est_aux.4
       CLR     est_aux.6                      ;el bit 6 de est_aux a 0
       CLR     est_aux.7                      ;el bit 7 de est_aux a 0
       CLR     est_aux.5                      ;el bit 5 de est_aux a 0
       ANL     est_cambio_sube,#00001111b     ;solo nos interesan los bits 0,1,2,3
       ANL     est_cambio_baja,#00001111b     ;solo nos interesan los bits 0,1,2,3
       MOV     A, est_aux
       XRL     A,est_cambio_sube        ;OR est_aux con est_cambio_sube para saber que bits son ascendentes
       XRL     A,est_cambio_baja        ;XOR est_aux con est_cambio_baja para saber que bits son descendentes
       MOV     est_aux,A
       ACALL   TABLA_PWM0              ;llama a la tabla de pwm0
       MOV     PWM0,A
       ACALL   TABLA1_PWM1              ;llama a la tabla1 de pwm1
       MOV     PWM1,A
       ACALL   TABLA2_PWM1
       ACALL   PWM1_AUX
       RET

PWM1_AUX:
    CJNE    A, #0x00, PWM1_AUX1
    CLR     P1.7
    CLR     P1.6        ; PWM1 parado
    RET

PWM1_AUX1:
    CJNE    A, #0x01, PWM1_AUX2
    CLR     P1.7
    SETB    P1.6        ; PWM1 izquierda
    RET

PWM1_AUX2:
    CJNE    A, #0x02, PWM1_AUX3
    SETB    P1.7
    CLR     P1.6        ; PWM1 derecha
    RET

PWM1_AUX3:
    SETB    P1.7
    SETB    P1.6        ; PWM1 avanza
    RET

TABLA_PWM0:
	; 1. TAULA - PWM0
	MOV A,est_aux
	INC A
	MOVC A,@A+PC
	RET
	DB 0xFF ; PWM0 %0-ra (geldi)
	DB 0xB3 ; %30 (jaitsi)
	DB 0x4D ; %70 (igo)
	NOP ; [-]
	DB 0x80 ; %50
	DB 0xB3 ; %30
	DB 0x4D ; %70
	NOP ; [-]
	DB 0x80 ; %50
	DB 0xB3 ; %30
	DB 0x4D ; %70
	NOP ; [-]
	NOP ; [-]
	NOP ; [-]
	NOP ; [-]
	NOP ; [-]
	DB 0x80 ; %50
	DB 0xB3 ; %30
	DB 0x4D ; %70
	NOP ; [-]
	DB 0x80 ; %50
	DB 0xB3 ; %30
	DB 0x4D ; %70
	NOP ; [-]
	DB 0x80 ; %50
	DB 0xB3 ; %30
	DB 0x4D ; %70
	NOP ; [-]
	NOP ; [-]
	NOP ; [-]
	NOP ; [-]
	NOP ; [-]

TABLA1_PWM1:
	; 2. TAULA - PWM1
	MOV A,est_aux
	INC A
	MOVC A,@A+PC
	RET
	DB 0xFF ; PWM1 %0-ra jarri (geldi)
	DB 0xFF ; %0
	DB 0xFF ; %0
	NOP ; [-]
	DB 0x80 ; %50
	DB 0x80 ; %50
	DB 0x80 ; %50
	NOP ; [-]
	DB 0x80 ; %50
	DB 0x80 ; %50
	DB 0x80 ; %50
	NOP ; [-]
	NOP ; [-]
	NOP ; [-]
	NOP ; [-]
	NOP ; [-]
	DB 0x80 ; %50
	DB 0x80 ; %50
	DB 0x80 ; %50
	NOP ; [-]
	DB 0x80 ; %50
	DB 0x80 ; %50
	DB 0x80 ; %50
	NOP ; [-]
	DB 0x80 ; %50
	DB 0x80 ; %50
	DB 0x80 ; %50
	NOP ; [-]
	NOP ; [-]
	NOP ; [-]
	NOP ; [-]
	NOP ; [-]

TABLA2_PWM1:
	; 3. TAULA - PWM1 CONTROL
	MOV A,est_aux
       INC A
	MOVC A,@A+PC
	RET
	DB 0x00 ; 00 (geldi)
	DB 0x00 ; 00
	DB 0x00 ; 00
	NOP ; [-]
	DB 0x02 ; 10 g_dcha
	DB 0x02 ; 10
	DB 0x02 ; 10
	NOP ; [-]
	DB 0x01 ; 01 g_izq
	DB 0x01 ; 01
	DB 0x01 ; 01
	NOP ; [-]
	NOP ; [-]
	NOP ; [-]
	NOP ; [-]
	NOP ; [-]
	DB 0x03 ; 11 (aurrera)
	DB 0x03 ; 11
	DB 0x03 ; 11
	NOP ; [-]
	DB 0x02 ; 10 g_dcha
	DB 0x02 ; 10
	DB 0x02 ; 10
	NOP ; [-]
	DB 0x01 ; 01 g_izq
	DB 0x01 ; 01
	DB 0x01 ; 01
	NOP ; [-]
	NOP ; [-]
	NOP ; [-]
	NOP ; [-]
	NOP ; [-]


EVENTOS3_6:
       ; tick_10s
       ;iniciar conversion bateria
       CLR    tick_10s
       ANL    ADCON,#11111000b
	ORL    ADCON,#0x02      ; canal P5.2 (bateria) y empieza conversion
       ORL 	ADCON,#08h	   ;Enciende el ADC 
       RET

EVENTOS3_8:
       ;tick_ADC_bat
       CLR    tick_ADC_bat
	ACALL  MOSTRAR_DISPLAY  ; mostrar valor en el display
       JB     bateria_baja,EVENTOS3_8_CONTINUA
       RET

EVENTOS3_8_CONTINUA:
       CLR    bateria_baja
       MOV    PWM0, #0xB3     ; PWM0 %30= bajar
      	MOV    PWM1, #0xFF     ; PWM1 %0= ni gira ni avanza
       MOV    ESTADO, #0x04   ; cambiar a estado ATERRIZAJE
       RET




;#################################################################################################################################
;                                                      Estado 4: ATERRIZAJE
;#################################################################################################################################

ATERRIZAJE:	
	ACALL 	MAQ_EVENTOS_4       	;Se llama a la máquina de eventos del Estado 4
	RET

MAQ_EVENTOS_4:
	MOV	A,EVENTO		;Trasladamos el valor del Evento actual al Acc
	RL	A			;Rotación a la izqu del valor del Acc (multiplicación por 2)
	MOV	DPTR,#LISTA_EV_4	;Mueve el dato al data pointer y nos dirige en la lista de eventos a donde queremos ir
	JMP	@A+DPTR		;Salto indirecto a la dir del dptr+Acc para colocarnos en el lugar de la lista correcto

LISTA_EV_4:
       RET                         
       NOP                  ;EVENTO 0
       RET                         
       NOP                  ;EVENTO 1
       RET                         
       NOP                  ;EVENTO 2
	AJMP	EVENTOS4_3	;EVENTO 3 tick_10ms
       RET                         
       NOP                  ;EVENTO 4
       RET                        
	NOP                  ;EVENTO 5 
       AJMP   EVENTOS4_6     ;EVENTO 6 tick_10s
       RET                        
	NOP                  ;EVENTO 7
       AJMP	EVENTOS4_8    ;EVENTO 8 tick_ADC_bat
       AJMP   EVENTOS4_9     ;EVENTO 9 tick_ADC_alt 
       RET                        
	NOP                  ;EVENTO 10
       RET                        
	NOP                  ;EVENTO 11
       RET

EVENTOS4_3:
       ;tick_10ms
       ;iniciar conversion altura
	ANL    ADCON,#11111000b      ; canal P5.0 (altura) y empieza a combertir
       ORL 	ADCON,#08h	        ;Enciende el ADC 
       RET

EVENTOS4_6:
       ; tick_10s
       ;iniciar conversion bateria
       CLR    tick_10s
       ANL    ADCON,#11111000b
	ORL    ADCON,#0x02      ; canal P5.2 (bateria) y empieza conversion
       ORL 	ADCON,#08h	   ;Enciende el ADC 
       RET

EVENTOS4_8:
       ;tick_ADC_bat
       CLR    tick_ADC_bat
	ACALL  MOSTRAR_DISPLAY ; mostrar valor en el display
       RET

EVENTOS4_9:
       ;tick_ADC_alt
       CLR    tick_ADC_alt
       MOV    A,ADCH
	SUBB   A,#0x01              ; 0.01m
	JC     EVENTOS4_9_CONTINUA; salta si carry=1 que significa que menor que 1cm
	RET

EVENTOS4_9_CONTINUA:
       MOV    PWM0,#0xFF    ;PWM0 0% parado
       MOV    PWM1,#0xFF    ;PWM1 0% parado
       CLR    P1.7
       CLR    P1.6
       CLR    desp_ater
       MOV    ESTADO, #0x00 ; cambiar a estado REPOSO
       RET


;#################################################################################################################################
;                                                      Estado 5: SUBIR
;#################################################################################################################################

SUBIR:	
	ACALL 	MAQ_EVENTOS_5       	;Se llama a la máquina de eventos del Estado 5
	RET

MAQ_EVENTOS_5:
	MOV	A,EVENTO		;Trasladamos el valor del Evento actual al Acc
	RL	A			;Rotación a la izqu del valor del Acc (multiplicación por 2)
	MOV	DPTR,#LISTA_EV_5	;Mueve el dato al data pointer y nos dirige en la lista de eventos a donde queremos ir
	JMP	@A+DPTR		;Salto indirecto a la dir del dptr+Acc para colocarnos en el lugar de la lista correcto

LISTA_EV_5:
       RET                         
       NOP                  ;EVENTO 0
       RET                         
       NOP                  ;EVENTO 1
       AJMP	EVENTOS5_2	;EVENTO 2 apagado
	AJMP	EVENTOS5_3	;EVENTO 3 tick_10ms
       RET                         
       NOP                  ;EVENTO 4
       RET                        
	NOP                  ;EVENTO 5 
       AJMP   EVENTOS5_6    ;EVENTO 6 tick_10s
       RET                        
	NOP                  ;EVENTO 7
       AJMP	EVENTOS5_8    ;EVENTO 8 tick_ADC_bat
       AJMP   EVENTOS5_9    ;EVENTO 9 tick_ADC_alt 
       RET                        
	NOP                  ;EVENTO 10
       RET                        
	NOP                  ;EVENTO 11
       RET


EVENTOS5_2:
       ;apagado
       MOV    PWM0, #0xB3     ; PWM0 %30= bajar
      	MOV    PWM1, #0xFF     ; PWM1 %0= ni gira ni avanza
       MOV    ESTADO, #0x04   ; cambiar a estado ATERRIZAJE
       RET

EVENTOS5_3:
       ;tick_10ms
       ;iniciar conversion altura
       CLR    tick_10ms
	ANL    ADCON,#11111000b      ; canal P5.0 (altura) y empieza a combertir
       ORL 	ADCON,#08h	        ;Enciende el ADC 
       RET

EVENTOS5_6:
       ; tick_10s
       ;iniciar conversion bateria
       CLR    tick_10s
       ANL    ADCON,#11111000b
	ORL    ADCON,#0x02      ; canal P5.2 (bateria) y empieza conversion
       ORL 	ADCON,#08h	   ;Enciende el ADC 
       RET

EVENTOS5_8:
       ;tick_ADC_bat
       CLR    tick_ADC_bat
	ACALL  MOSTRAR_DISPLAY  ; mostrar valor en el display
       JB     bateria_baja,EVENTOS5_8_CONTINUA
       RET

EVENTOS5_8_CONTINUA:
       CLR    bateria_baja
       MOV    PWM0, #0xB3     ; PWM0 %30= bajar
      	MOV    PWM1, #0xFF     ; PWM1 %0= ni gira ni avanza
       MOV    ESTADO, #0x04   ; cambiar a estado ATERRIZAJE
       RET

EVENTOS5_9:
       ;tick_ADC_alt
       CLR    tick_ADC_alt
	MOV    A,ADCH         ; mover el valor convertido al acumulador
	SUBB   A,#0x55        
	JC     EVENTOS5_9_CONTINUA ; comparar con 1m
       MOV    PWM1,#0x80	 ; ha llegado a 1 metro
       SETB   P1.7
       SETB   P1.6           ; avanza
	MOV    PWM0,#0x80     ; PWM0 al %50 mantiene altura
       MOV    ESTADO,#0x02   ; cambia a VUELO ESTABLE
	RET

EVENTOS5_9_CONTINUA:
       RET


;#################################################################################################################################
;                                                      Estado 6: BAJAR
;#################################################################################################################################

BAJAR:	
	ACALL 	MAQ_EVENTOS_6       	;Se llama a la máquina de eventos del Estado 6
	RET

MAQ_EVENTOS_6:
	MOV	A,EVENTO		;Trasladamos el valor del Evento actual al Acc
	RL	A			;Rotación a la izqu del valor del Acc (multiplicación por 2)
	MOV	DPTR,#LISTA_EV_6	;Mueve el dato al data pointer y nos dirige en la lista de eventos a donde queremos ir
	JMP	@A+DPTR		;Salto indirecto a la dir del dptr+Acc para colocarnos en el lugar de la lista correcto


LISTA_EV_6:
       RET                         
       NOP                  ;EVENTO 0
       RET                         
       NOP                  ;EVENTO 1
       AJMP	EVENTOS6_2	;EVENTO 2 apagado
	AJMP	EVENTOS6_3	;EVENTO 3 tick_10ms
       RET                         
       NOP                  ;EVENTO 4
       RET                        
	NOP                  ;EVENTO 5 
       AJMP   EVENTOS6_6    ;EVENTO 6 tick_10s
       RET                        
	NOP                  ;EVENTO 7
       AJMP	EVENTOS6_8    ;EVENTO 8 tick_ADC_bat
       AJMP   EVENTOS6_9    ;EVENTO 9 tick_ADC_alt 
       RET                        
	NOP                  ;EVENTO 10
       RET                        
	NOP                  ;EVENTO 11
       RET


EVENTOS6_2:
       ;apagado
       MOV    PWM0, #0xB3     ; PWM0 %30= bajar
      	MOV    PWM1, #0xFF     ; PWM1 %0= ni gira ni avanza
       MOV    ESTADO, #0x04   ; cambiar a estado ATERRIZAJE
       RET

EVENTOS6_3:
       ;tick_10ms
       ;iniciar conversion altura
       CLR    tick_10ms
	ANL    ADCON,#11111000b      ; canal P5.0 (altura) y empieza a combertir
       ORL 	ADCON,#08h	        ;Enciende el ADC 
       RET

EVENTOS6_6:
       ; tick_10s
       ;iniciar conversion bateria
       CLR    tick_10s
       ANL    ADCON,#11111000b
	ORL    ADCON,#0x02      ; canal P5.2 (bateria) y empieza conversion
       ORL 	ADCON,#08h	   ;Enciende el ADC 
       RET

EVENTOS6_8:
       ;tick_ADC_bat
       CLR    tick_ADC_bat
	ACALL  MOSTRAR_DISPLAY  ; mostrar valor en el display
       JB     bateria_baja,EVENTOS6_8_CONTINUA
       RET

EVENTOS6_8_CONTINUA:
       CLR    bateria_baja
       MOV    PWM0, #0xB3     ; PWM0 %30= bajar
      	MOV    PWM1, #0xFF     ; PWM1 %0= ni gira ni avanza
       MOV    ESTADO, #0x04   ; cambiar a estado ATERRIZAJE
       RET

EVENTOS6_9:
       ;tick_ADC_alt
       CLR    tick_ADC_alt
	MOV    A,ADCH         ; mover el valor convertido al acumulador
	SUBB   A,#0x55        
	JC     EVENTOS6_9_CONTINUA ; comparar con 1m
       MOV    PWM1,#0x80	 ; ha llegado a 1 metro
       SETB   P1.7
       SETB   P1.6           ; avanza
	MOV    PWM0,#0x80     ; PWM0 al %50 mantiene altura
       MOV    ESTADO,#0x02   ; cambia a VUELO ESTABLE
	RET

EVENTOS6_9_CONTINUA:
       RET


;#################################################################################################################################
;                                                      Estado 7: GIRO_DCHA (por obstaculo)
;#################################################################################################################################

GIRO_DCHA:	
	ACALL 	MAQ_EVENTOS_7       	;Se llama a la máquina de eventos del Estado 7
	RET

MAQ_EVENTOS_7:
	MOV	A,EVENTO		;Trasladamos el valor del Evento actual al Acc
	RL	A			;Rotación a la izqu del valor del Acc (multiplicación por 2)
	MOV	DPTR,#LISTA_EV_7	;Mueve el dato al data pointer y nos dirige en la lista de eventos a donde queremos ir
	JMP	@A+DPTR		;Salto indirecto a la dir del dptr+Acc para colocarnos en el lugar de la lista correcto


LISTA_EV_7:
       RET                         
       NOP                  ;EVENTO 0
       RET                         
       NOP                  ;EVENTO 1
       AJMP	EVENTOS7_2	;EVENTO 2 apagado
	AJMP	EVENTOS7_3	;EVENTO 3 tick_10ms
       RET                         
       NOP                  ;EVENTO 4
       AJMP	EVENTOS7_5	;EVENTO 5 tick_8ms
       AJMP   EVENTOS7_6    ;EVENTO 6 tick_10s
       RET                        
	NOP                  ;EVENTO 7
       AJMP	EVENTOS7_8    ;EVENTO 8 tick_ADC_bat
       RET                        
	NOP                  ;EVENTO 9
       AJMP   EVENTOS7_10   ;EVENTO 10 tick_ADC_frnt
       RET                        
	NOP                  ;EVENTO 11
       RET


EVENTOS7_2:
       ;apagado
       MOV    PWM0, #0xB3     ; PWM0 %30= bajar
      	MOV    PWM1, #0xFF     ; PWM1 %0= ni gira ni avanza
       MOV    ESTADO, #0x04   ; cambiar a estado ATERRIZAJE
       RET

EVENTOS7_3:
       ;tick_10ms
       ;iniciar conversion frente
       CLR    tick_10ms
       ANL    ADCON,#11111000b
	ORL    ADCON,#0x01      ; canal P5.1 (frente) 
       ORL 	ADCON,#08h	   ;Enciende el ADC 
       RET

EVENTOS7_5:
       ;tick_8s
       CLR    tick_8s
       MOV    PWM0, #0xB3     ; PWM0 %30= bajar
      	MOV    PWM1, #0xFF     ; PWM1 %0= ni gira ni avanza
       MOV    ESTADO, #0x04   ; cambiar a estado ATERRIZAJE
       RET

EVENTOS7_6:
       ;tick_10s
       ;iniciar conversion bateria
       CLR    tick_10s
       ANL    ADCON,#11111000b
	ORL    ADCON,#0x02      ; canal P5.2 (bateria) 
       ORL 	ADCON,#08h	   ;Enciende el ADC 
       RET

EVENTOS7_8:
       ;tick_ADC_bat
       CLR    tick_ADC_bat
	ACALL  MOSTRAR_DISPLAY  ; mostrar valor en el display
       JB     bateria_baja,EVENTOS7_8_CONTINUA
       RET

EVENTOS7_8_CONTINUA:
       CLR    bateria_baja
       MOV    PWM0, #0xB3     ; PWM0 %30= bajar
      	MOV    PWM1, #0xFF     ; PWM1 %0= ni gira ni avanza
       MOV    ESTADO, #0x04   ; cambiar a estado ATERRIZAJE
       RET

EVENTOS7_10:
       ;tick_ADC_frnt
       MOV   A,ADCH  
	SUBB  A,#0x44             ; 0.8m
	JNC   EVENTOS7_10_CONTINUA ;salta si carry=0 que significa que mayor o igual que 80cm
	RET

EVENTOS7_10_CONTINUA:
       SETB   P1.7
       SETB   P1.6               ; PWM1 avanza
       MOV    ESTADO,#0x02       ; cambia a VUELO ESTABLE
       RET

        


       
;#################################################################################################################################
;                                                      Estado 8: GIRO_IZQ (por timeout)
;#################################################################################################################################

GIRO_IZQ:	
	ACALL 	MAQ_EVENTOS_8       	;Se llama a la máquina de eventos del Estado 8
	RET

MAQ_EVENTOS_8:
	MOV	A,EVENTO		;Trasladamos el valor del Evento actual al Acc
	RL	A			;Rotación a la izqu del valor del Acc (multiplicación por 2)
	MOV	DPTR,#LISTA_EV_8	;Mueve el dato al data pointer y nos dirige en la lista de eventos a donde queremos ir
	JMP	@A+DPTR		;Salto indirecto a la dir del dptr+Acc para colocarnos en el lugar de la lista correcto



LISTA_EV_8:
       RET                         
       NOP                  ;EVENTO 0
       RET                         
       NOP                  ;EVENTO 1
       AJMP	EVENTOS8_2	;EVENTO 2 apagado
       RET                         
       NOP                  ;EVENTO 3
       AJMP	EVENTOS8_4	;EVENTO 4 tick_500ms 
       RET                         
       NOP                  ;EVENTO 5
       AJMP	EVENTOS8_6	;EVENTO 6 tick_10s
       RET                         
       NOP                  ;EVENTO 7
       AJMP	EVENTOS8_8	;EVENTO 8 tick_ADC_bat
       RET                         
       NOP                  ;EVENTO 9
       RET                         
       NOP                  ;EVENTO 10
       RET                         
       NOP                  ;EVENTO 11
       RET


EVENTOS8_2:
       ;apagado
       MOV    PWM0, #0xB3     ; PWM0 %30= bajar
      	MOV    PWM1, #0xFF     ; PWM1 %0= ni gira ni avanza
       MOV    ESTADO, #0x04   ; cambiar a estado ATERRIZAJE
       RET

EVENTOS8_4:
       ;tick_500ms
       CLR    tick_500ms
       SETB   P1.7
       SETB   P1.6               ; PWM1 avanza
       MOV    ESTADO,#0x02       ; cambia a VUELO ESTABLE
       RET

EVENTOS8_6:
       ;tick_10s
       ;iniciar conversion bateria
       CLR    tick_10s
       ANL    ADCON,#11111000b
	ORL    ADCON,#0x02      ; canal P5.2 (bateria) 
       ORL 	ADCON,#08h	   ;Enciende el ADC 
       RET

EVENTOS8_8:
       ;tick_ADC_bat
       CLR    tick_ADC_bat
	ACALL  MOSTRAR_DISPLAY  ; mostrar valor en el display
       JB     bateria_baja,EVENTOS8_8_CONTINUA
       RET

EVENTOS8_8_CONTINUA:
       CLR    bateria_baja
       MOV    PWM0, #0xB3     ; PWM0 %30= bajar
      	MOV    PWM1, #0xFF     ; PWM1 %0= ni gira ni avanza
       MOV    ESTADO, #0x04   ; cambiar a estado ATERRIZAJE
       RET
       

	







;#################################################################################################################################
;                                                      SUBRRUTINAS
;#################################################################################################################################

;************************************************      PROGRAMAR TIMER     **************************************************

PROGRAM_TIMER:				
	MOV	TH0,#0XB1		;Valor Adjudicado al TH0 del Timer
	MOV	TL0,#0XDF		;Valor Adjudicado al TL0 del Timer
	MOV	cont_10ms,#0		;SE INICIALIZAN LAS VARIABLES DE CONTADORES A 0    
       MOV    cont_500ms_8s,#0	;SE INICIALIZAN LAS VARIABLES DE CONTADORES A 0  
       MOV    cont_500ms_10s,#0	;SE INICIALIZAN LAS VARIABLES DE CONTADORES A 0   
       MOV    cont_500ms_25s,#0	;SE INICIALIZAN LAS VARIABLES DE CONTADORES A 0  
	CLR	tick_10ms		;SE BORRAN TODOS LOS TICKS DE TIMER
	CLR	tick_500ms		;SE BORRAN TODOS LOS TICKS DE TIMER
	CLR	tick_8s		;SE BORRAN TODOS LOS TICKS DE TIMER
	CLR 	tick_10s		;SE BORRAN TODOS LOS TICKS DE TIMER
       CLR	tick_25s		;SE BORRAN TODOS LOS TICKS DE TIMER
	MOV	TMOD,#00000001   	;timer 0 en modo 1
       SETB	TCON.4			;ENCIENDE EL TIMER0
       SETB	ET0			;ACTIVACIÓN DE INTERRUPCIÓN TIMER0 A8.1 IE.1
	SETB	EA			;SE ACTIVA EL BIT QUE ACTIVA LAS INTERRUPCIONES(ENABLE ALL)
	RET


;************************************************      VISUALIZAR DISPLAY      **************************************************

MOSTRAR_DISPLAY: 
	MOV   BATERIA,ADCH

DISPLAY_L:
	MOV   A,BATERIA
	SUBB  A,#0x01
	JNC   DISPLAY_1     ;salta si carry=0 que significa que mayor o igual
	MOV   A,#0x00
       SETB  bateria_baja 
	AJMP  FIN_DISPLAY
DISPLAY_1:
	MOV   A,BATERIA
	SUBB  A,#0x03
	JNC   DISPLAY_2
	MOV   A,#0x01
	AJMP  FIN_DISPLAY
DISPLAY_2:
	MOV   A,BATERIA
	SUBB  A,#0x06
	JNC   DISPLAY_3
	MOV   A,#0x02
	AJMP  FIN_DISPLAY
DISPLAY_3:
	MOV   A,BATERIA
	SUBB  A,#0x08
	JNC   DISPLAY_4
	MOV   A,#0x03
	AJMP  FIN_DISPLAY
DISPLAY_4:
	MOV   A,BATERIA
	SUBB  A,#0x0B
	JNC   DISPLAY_5
	MOV   A,#0x04
	AJMP  FIN_DISPLAY
DISPLAY_5:
	MOV   A,BATERIA
	SUBB  A,#0x0D
	JNC   DISPLAY_6
	MOV   A,#0x05
	AJMP  FIN_DISPLAY
DISPLAY_6:
	MOV   A,BATERIA
	SUBB  A,#0x10
	JNC   DISPLAY_7
	MOV   A,#0x06
	AJMP  FIN_DISPLAY
DISPLAY_7:
	MOV   A,BATERIA
	SUBB  A,#0x12
	JNC   DISPLAY_8
	MOV   A,#0x07
	AJMP  FIN_DISPLAY
DISPLAY_8:
	MOV   A,BATERIA
	SUBB  A,#0x15
	JNC   DISPLAY_9
	MOV   A,#0x08
	AJMP  FIN_DISPLAY
DISPLAY_9:
	MOV   A,BATERIA
	SUBB  A,#0x17
	JNC   DISPLAY_F
	MOV   A,#0x09
	AJMP  FIN_DISPLAY
DISPLAY_F:
	MOV   A,#0x0A

FIN_DISPLAY:
	ACALL TABLA_DISPLAY
	MOV   C,DISPLAY.7
	MOV   DISPLAY,A
	MOV   DISPLAY.7,C
	ACALL SUB_LED
	RET

TABLA_DISPLAY:
	INC A
	MOVC A,@A+PC
	RET
	DB 10111000b ; (L) 
	DB 10000110b ; 1
	DB 11011011b ; 2
	DB 11001111b ; 3
	DB 11100110b ; 4
	DB 11101101b ; 5
	DB 11111101b ; 6
	DB 10000111b ; 7
	DB 11111111b ; 8
	DB 11101111b ; 9
	DB 11110001b ; F
	RET

SUB_LED:
       CLR   C
	MOV   A,BATERIA
	SUBB  A,#0x02
	JC    SUB_LED_BATERIA_BAJA
	SETB  LED_verde
	CLR   LED_rojo
	RET

SUB_LED_BATERIA_BAJA:
	CLR   LED_verde
	SETB  LED_rojo
	RET


;#################################################################################################################################
;                                                      INTERRUPCIONES
;#################################################################################################################################	

;**********************************************    ADC  **************************************************************

SUB_ADC:
       ANL   ADCON, #11101111b	; ADCI=0 
	MOV   A,ADCON               ; 
	ANL   A,#0x03
	RL A
	RL A
	MOV DPTR,#ADC_TIPO
	JMP @A+DPTR
ADC_TIPO:
	SETB tick_ADC_alt
	RETI
	NOP
	SETB tick_ADC_frnt
	RETI
	NOP
	SETB tick_ADC_bat
	RETI 


;**********************************************    TIMER  **************************************************************
SUB_TIMER:
       PUSH      ACC
       PUSH      PSW
       MOV       TH0,#0XB1
       MOV       TL0,#0XDF
       SETB      tick_10ms
       INC       cont_10ms
       MOV       A,cont_10ms
       CLR       C
       SUBB      A,#0x32
       JC        FIN_SUBTIMER
       MOV       cont_10ms,#0
       SETB      tick_500ms
       INC       cont_500ms_8s
       INC       cont_500ms_10S
       INC       cont_500ms_25S
       MOV       A,cont_500ms_8s
       CJNE      A,#0x10,SUB_TIMER_CONTINUA1 ; salta si no han pasado 8s
       MOV       cont_500ms_8s,#0
       SETB      tick_8s

SUB_TIMER_CONTINUA1:
       MOV   A,cont_500ms_10s
	CJNE  A,#0x14,SUB_TIMER_CONTINUA2 ; salta si no han pasado 10s
	MOV   cont_500ms_10s,#0
	SETB  tick_10s

SUB_TIMER_CONTINUA2:
	MOV   A,cont_500ms_25s
	CJNE  A,#0x32,FIN_SUBTIMER ; salta si no han pasado 25s
	MOV   cont_500ms_25s,#0
	SETB  tick_25s


FIN_SUBTIMER:
       POP PSW
       POP ACC
       RET
       
END