ORG 00H
; INITIALIZE SERIAL COMMUNICATION
MOV TMOD,#20H ;Timer 1, mode 2 ; 0010 0000 ;  8 bit auto reload
MOV TH1,#0FDH ;9600 baud rate
MOV SCON,#50H; 8 bit data,1 stop bit, 1 start bit, REN enabled ; 0101 0000 
CLR T1 ; Clear Timer 1 register
SETB TR1 ;start timer 1

;PIN CONFIGURATION
LED_1     BIT P1.0 ; the buffer port is port 1. 
LED_2     BIT P1.1
LED_3     BIT P1.2
LED_4     BIT P1.3
LED_5     BIT P1.4
LED_6     BIT P1.5
LED_7     BIT P1.6
LED_8     BIT P1.7

LED_RELAY BIT P2.5

CLR  LED_1
CLR  LED_2
CLR  LED_3
CLR  LED_4
CLR  LED_5
CLR  LED_6
CLR  LED_7
CLR  LED_8
SETB LED_RELAY

RS   EQU P2.0
RW   EQU P2.1
ENBL EQU P2.2

;SEND DATA FROM MICROCONTROLLER TO SMARTPHONE 
MOV DPTR, #MYDATA 
GO: 
    CLR A    
    MOVC A,@A+DPTR
    JZ LCD ; LCD Subroutine
    ACALL SEND
    SJMP GO

SEND:
    MOV SBUF, A 
    INC DPTR
    HERE: JNB TI, HERE ;wait for the last bit to transfer
    CLR TI ;clear TI for the next INPUT
    RET

;MAKING THE LCD READY FOR DISPLAYING DATA FROM SMARTPHONE

; LCD Sub routine
LCD: 
    MOV SP,#70H ; 112D of ROM
    MOV PSW,#00H

MOV A,#38H ;LCD 2 lines,5 by 7 matrix
LCALL COMMAND ;call command subroutine
LCALL DELAY ;give LCD some time

MOV A,#0EH ;display on, cursor on
LCALL COMMAND
LCALL DELAY

WELCOME:
MOV 32H, #16
LCALL CLEAR_LCD

MOV A,#06H ;shift cursor right
LCALL COMMAND
LCALL DELAY

MOV A,#80H ;force cursor to begin at 1st line
LCALL COMMAND
LCALL DELAY

    ;DISPLAY WELCOME MESSAGE
    LCALL DELAY
    ; Display the MYDATA string
    MOV DPTR, #MYDATA ; Load the address of the string
LOOP_1: CLR A
    MOVC A,@A+DPTR
    JZ FINISH
    LCALL DISPLAY
    LCALL DELAY
    INC DPTR
    LJMP LOOP_1

FINISH: 
    LJMP SELECT_MODE

MSG_A: DB '-.', 0     ; .-
MSG_B: DB '-...', 0   ; -...
MSG_C: DB '-.-.', 0   ; -.-.
MSG_D: DB '-..', 0   ; -..
MSG_E: DB '.', 0   ; .
MSG_F: DB '..-.', 0   ; ..-.
MSG_G: DB '--.', 0   ; --.
MSG_H: DB '....', 0   ; ....
MSG_I: DB '..', 0   ; ..
MSG_J: DB '.---', 0   ; .---
MSG_K: DB '-.-', 0   ; -.-
MSG_L: DB '.-..', 0   ; .-..
MSG_M: DB '--', 0   ; --
MSG_N: DB '-.', 0   ; -.
MSG_O: DB '---', 0   ; ---
MSG_P: DB '.--.', 0   ; .--.
MSG_Q: DB '--.-', 0   ; --.-
MSG_R: DB '.-.', 0   ; .-.
MSG_S: DB '...', 0   ; ...
MSG_T: DB '-', 0   ; -
MSG_U: DB '..-', 0   ; ..-
MSG_V: DB '...-', 0   ; ...-
MSG_W: DB '.--', 0   ; .--
MSG_X: DB '-..', '-', 0   ; -..-
MSG_Y: DB '-.--', 0   ; -.-- 
MSG_Z: DB '--..', 0   ; --..


MYDATA: DB 'B2 Group- 4',0 ; this data is shown from virtual terminal
MSG1: DB 'ENTER MODE:',0


SELECT_MODE:
MOV 32H, #16
LCALL CLEAR_LCD
MOV DPTR, #MSG1 ; Load the address of the string
LOOP_A: CLR A
MOVC A,@A+DPTR
JZ FINISH1
LCALL DISPLAY
LCALL DELAY
INC DPTR
LJMP LOOP_A

FINISH1:
ACALL GET_IP
CJNE A, #'1',NEXT1
LJMP MODE1
NEXT1: CJNE A, #'2',NEXT2
LJMP  MODE2
NEXT2: CJNE A, #'3',NEXT3
LJMP MODE3
NEXT3: CJNE A, #'4',NEXT4
LJMP MODE4
NEXT4: CJNE A, #'5',NEXT5
LJMP MODE5
NEXT5: CJNE A, #'6',NEXT5
LJMP MODE6

;COMMAND FROM THE SMARTPHONE
MODE1:
    MOV 32H, #16
    LCALL CLEAR_LCD

INPUT_NEW: 
    LCALL GET_IP
    CJNE A,#'1', CHECK1
    CPL LED_1;turn on the LED
    LJMP INPUT_NEW ;print the INPUT_NEW_NEWacter pressed


CHECK1: CJNE A, #'2', CHECK2
CPL LED_2
LJMP INPUT_NEW

CHECK2: CJNE A, '3', CHECK3
CPL LED_3
LJMP INPUT_NEW

CHECK3: CJNE A, #'4', CHECK4
CPL LED_4
LJMP INPUT_NEW

CHECK4: CJNE A, #'5', CHECK5
CPL LED_5
LJMP INPUT_NEW

CHECK5: CJNE A, #'6', CHECK6
CPL LED_6
LJMP INPUT_NEW

CHECK6: CJNE A, #'7', CHECK7
CPL LED_7
LJMP INPUT_NEW

CHECK7: CJNE A, #'8', CHECK8
CPL LED_8
LJMP INPUT_NEW

CHECK8: CJNE A, #'0', INPUT_NEW
clr p1
acall delay 
LJMP SELECT_MODE


MODE2:
MOV 32H, #16
LCALL CLEAR_LCD
ACALL GET_IP

clr p2.5
acall delay

setb p2.5 
doo: cjne a, #'0', doo 

END_MODE2: LJMP SELECT_MODE


MODE3:  
MOV 32H, #16
LCALL CLEAR_LCD

CHECK_A:
    ACALL GET_IP
    CJNE A, #'A', CHECK_B
    MOV DPTR, #MSG_A
    LJMP DISPLAY_MORSE

CHECK_B:
    MOV A, SBUF
    CJNE A, #'B', CHECK_C
    MOV DPTR, #MSG_B
    LJMP DISPLAY_MORSE

CHECK_C:
    MOV A, SBUF
    CJNE A, #'C', CHECK_D
    MOV DPTR, #MSG_C
    LJMP DISPLAY_MORSE

CHECK_D:
    MOV A, SBUF
    CJNE A, #'D', CHECK_E
    MOV DPTR, #MSG_D
    LJMP DISPLAY_MORSE

CHECK_E:
    MOV A, SBUF
    CJNE A, #'E', CHECK_F
    MOV DPTR, #MSG_E
    LJMP DISPLAY_MORSE

CHECK_F:
    MOV A, SBUF
    CJNE A, #'F', CHECK_G
    MOV DPTR, #MSG_F
    LJMP DISPLAY_MORSE

CHECK_G:
    MOV A, SBUF
    CJNE A, #'G', CHECK_H
    MOV DPTR, #MSG_G
    LJMP DISPLAY_MORSE

CHECK_H:
    MOV A, SBUF
    CJNE A, #'H', CHECK_I
    MOV DPTR, #MSG_H
    LJMP DISPLAY_MORSE

CHECK_I:
    MOV A, SBUF
    CJNE A, #'I', CHECK_J
    MOV DPTR, #MSG_I
    LJMP DISPLAY_MORSE

CHECK_J:
    MOV A, SBUF
    CJNE A, #'J', CHECK_K
    MOV DPTR, #MSG_J
    LJMP DISPLAY_MORSE

CHECK_K:
    MOV A, SBUF
    CJNE A, #'K', CHECK_L
    MOV DPTR, #MSG_K
    LJMP DISPLAY_MORSE

CHECK_L:
    MOV A, SBUF
    CJNE A, #'L', CHECK_M
    MOV DPTR, #MSG_L
    LJMP DISPLAY_MORSE

CHECK_M:
    MOV A, SBUF
    CJNE A, #'M', CHECK_N
    MOV DPTR, #MSG_M
    LJMP DISPLAY_MORSE

CHECK_N:
    MOV A, SBUF
    CJNE A, #'N', CHECK_O
    MOV DPTR, #MSG_N
    LJMP DISPLAY_MORSE

CHECK_O:
    MOV A, SBUF
    CJNE A, #'O', CHECK_P
    MOV DPTR, #MSG_O
    LJMP DISPLAY_MORSE

CHECK_P:
    MOV A, SBUF
    CJNE A, #'P', CHECK_Q
    MOV DPTR, #MSG_P
    LJMP DISPLAY_MORSE

CHECK_Q:
    MOV A, SBUF
    CJNE A, #'Q', CHECK_R
    MOV DPTR, #MSG_Q
    LJMP DISPLAY_MORSE

CHECK_R:
    MOV A, SBUF
    CJNE A, #'R', CHECK_S
    MOV DPTR, #MSG_R
    LJMP DISPLAY_MORSE

CHECK_S:
    MOV A, SBUF
    CJNE A, #'S', CHECK_T
    MOV DPTR, #MSG_S
    LJMP DISPLAY_MORSE

CHECK_T:
    MOV A, SBUF
    CJNE A, #'T', CHECK_U
    MOV DPTR, #MSG_T
    LJMP DISPLAY_MORSE

CHECK_U:
    MOV A, SBUF
    CJNE A, #'U', CHECK_V
    MOV DPTR, #MSG_U
    LJMP DISPLAY_MORSE

CHECK_V:
    MOV A, SBUF
    CJNE A, #'V', CHECK_W
    MOV DPTR, #MSG_V
    LJMP DISPLAY_MORSE

CHECK_W:
    MOV A, SBUF
    CJNE A, #'W', CHECK_X
    MOV DPTR, #MSG_W
    LJMP DISPLAY_MORSE

CHECK_X:
    MOV A, SBUF
    CJNE A, #'X', CHECK_Y
    MOV DPTR, #MSG_X
    LJMP DISPLAY_MORSE

CHECK_Y:
    MOV A, SBUF
    CJNE A, #'Y', CHECK_Z
    MOV DPTR, #MSG_Y
    LJMP DISPLAY_MORSE

CHECK_Z:
    MOV A, SBUF
    CJNE A, #'Z', NEXT6
    MOV DPTR, #MSG_Z
    LJMP DISPLAY_MORSE

NEXT6: MOV A,SBUF
CJNE A, #'0', FINISH2
LJMP SELECT_MODE

FINISH2: LJMP CHECK_A

DISPLAY_MORSE:
    LOOP_MORSE: 
        CLR A
        MOVC A, @A+DPTR
        JNZ NOT_ZERO_MORSE
        ZERO_MORSE: LJMP CHECK_A

    NOT_ZERO_MORSE:
        LCALL DISPLAY
        LCALL DELAY
        INC DPTR
        LJMP LOOP_MORSE

MODE4:

ENCRYPTION:
    MOV 32H, #16
    LCALL CLEAR_LCD

    ENCRYPTION_INPUT:
     LCALL GET_IP

        CJNE A, #'Z', NOT_Z_ENCRYPTION
        MOV A, #'A' ; EQUAL TO Z
        LCALL DISPLAY
        LCALL DELAY
        LJMP ENCRYPTION_INPUT

        NOT_Z_ENCRYPTION:
            MOV A, SBUF
            CJNE A, #'0', CONTINUE_ENCRYPTION
            LJMP SELECT_MODE ; STOP ENCRYPTION AND GO BACK TO MAIN INPUT SUBROUTINE
    
        CONTINUE_ENCRYPTION:
            ADD A, #1
            LCALL DISPLAY
            LCALL DELAY
            LJMP ENCRYPTION_INPUT

MODE5:
    MOV 32H, #16
    LCALL CLEAR_LCD

    MOV R0, #50H
    
    INPUT_NEW_MORSE:
    MOV R4 , #0
    MOV R6 , #0
    MOV R5 , #0
    MOV R7 , #0

    MOV R3 , #1 ;COUNTER
    
    MORSE_DECRYPT_INPUT: 
        LCALL GET_IP
        ACALL DISPLAY
        ACALL DELAY
        CJNE A, #' ', CONTINUE_MORSE_IP
        LJMP MORSE_DECRYPT
        
    CONTINUE_MORSE_IP:
    	CJNE A, #'/', NEXT20
        LJMP DISPLAY_CHAR

        NEXT20: 
        CJNE R3 , #1 , NOT_1 ; if R5=1, store the input in R4
        SJMP COUNTER_1
    
        NOT_1:
        CJNE R3 , #2 , NOT_2 ; if R5=2, store the input in R5
        SJMP COUNTER_2

        NOT_2:
        CJNE R3 , #3 , NOT_3
        SJMP COUNTER_3

        NOT_3:
        CJNE R3 , #4 , NOT_4
        SJMP COUNTER_4

        NOT_4:
        LJMP MODE5

        COUNTER_1:
        MOV R4 , A
        INC R3
        LJMP MORSE_DECRYPT_INPUT
        
        COUNTER_2:
        MOV R5 , A
        INC R3
        LJMP MORSE_DECRYPT_INPUT
        
        COUNTER_3:
        MOV R6, A
        INC R3
        LJMP MORSE_DECRYPT_INPUT

        COUNTER_4:
        MOV R7 , A
        LJMP MORSE_DECRYPT_INPUT


    MORSE_DECRYPT:
        ;CHECK FOR A
        CJNE R4, #'.' , NOT_MORSE_A
        CJNE R5, #'-' , NOT_MORSE_A
        CJNE R6, #0 , NOT_MORSE_A
        CJNE R7, #0 , NOT_MORSE_A
        LJMP MORSE_A
        
        ;CHECK FOR B
        NOT_MORSE_A: 
        CJNE R4, #'-' , NOT_MORSE_B
        CJNE R5, #'.' , NOT_MORSE_B
        CJNE R6, #'.' , NOT_MORSE_B
        CJNE R7, #'.' , NOT_MORSE_B
        LJMP MORSE_B

        ; CHECK FOR C
        NOT_MORSE_B:
        CJNE R4, #'-' , NOT_MORSE_C
        CJNE R5, #'.' , NOT_MORSE_C
        CJNE R6, #'-' , NOT_MORSE_C
        CJNE R7, #'.' , NOT_MORSE_C
        LJMP MORSE_C

        ; CHECK FOR D
        NOT_MORSE_C:
        CJNE R4, #'-' , NOT_MORSE_D
        CJNE R5, #'.' , NOT_MORSE_D
        CJNE R6, #'.' , NOT_MORSE_D
        CJNE R7, #0 , NOT_MORSE_D
        LJMP MORSE_D

        ; CHECK FOR E
        NOT_MORSE_D:
        CJNE R4, #'.' , NOT_MORSE_E
        CJNE R5, #0 , NOT_MORSE_E
        CJNE R6, #0 , NOT_MORSE_E
        CJNE R7, #0 , NOT_MORSE_E
        LJMP MORSE_E

        ; CHECK FOR F
        NOT_MORSE_E:
        CJNE R4, #'.' , NOT_MORSE_F
        CJNE R5, #'.' , NOT_MORSE_F
        CJNE R6, #'-' , NOT_MORSE_F
        CJNE R7, #'.' , NOT_MORSE_F
        LJMP MORSE_F

        ; CHECK FOR G
        NOT_MORSE_F:
        CJNE R4, #'-' , NOT_MORSE_G
        CJNE R5, #'-' , NOT_MORSE_G
        CJNE R6, #'.' , NOT_MORSE_G
        CJNE R7, #0 , NOT_MORSE_G
        LJMP MORSE_G

        ; CHECK FOR H
        NOT_MORSE_G:
        CJNE R4, #'.' , NOT_MORSE_H
        CJNE R5, #'.' , NOT_MORSE_H
        CJNE R6, #'.' , NOT_MORSE_H
        CJNE R7, #'.' , NOT_MORSE_H
        LJMP MORSE_H

        ; CHECK FOR I
        NOT_MORSE_H:
        CJNE R4, #'.' , NOT_MORSE_I
        CJNE R5, #'.' , NOT_MORSE_I
        CJNE R6, #0 , NOT_MORSE_I
        CJNE R7, #0 , NOT_MORSE_I
        LJMP MORSE_I

        ; CHECK FOR J
        NOT_MORSE_I:
        CJNE R4, #'.' , NOT_MORSE_J
        CJNE R5, #'-' , NOT_MORSE_J
        CJNE R6, #'-' , NOT_MORSE_J
        CJNE R7, #'-' , NOT_MORSE_J
        LJMP MORSE_J

        ; CHECK FOR K
        NOT_MORSE_J:
        CJNE R4, #'-' , NOT_MORSE_K
        CJNE R5, #'.' , NOT_MORSE_K
        CJNE R6, #'-' , NOT_MORSE_K
        CJNE R7, #0 , NOT_MORSE_K
        LJMP MORSE_K

        ; CHECK FOR L
        NOT_MORSE_K:
        CJNE R4, #'.' , NOT_MORSE_L
        CJNE R5, #'-' , NOT_MORSE_L
        CJNE R6, #'.' , NOT_MORSE_L
        CJNE R7, #'.' , NOT_MORSE_L
        LJMP MORSE_L

        ; CHECK FOR M
        NOT_MORSE_L:
        CJNE R4, #'-' , NOT_MORSE_M
        CJNE R5, #'-' , NOT_MORSE_M
        CJNE R6, #0 , NOT_MORSE_M
        CJNE R7, #0 , NOT_MORSE_M
        LJMP MORSE_M

        ; CHECK FOR N
        NOT_MORSE_M:
        CJNE R4, #'-' , NOT_MORSE_N
        CJNE R5, #'.' , NOT_MORSE_N
        CJNE R6, #0 , NOT_MORSE_N
        CJNE R7, #0 , NOT_MORSE_N
        LJMP MORSE_N

        ; CHECK FOR O
        NOT_MORSE_N:
        CJNE R4, #'-' , NOT_MORSE_O
        CJNE R5, #'-' , NOT_MORSE_O
        CJNE R6, #'-' , NOT_MORSE_O
        CJNE R7, #0 , NOT_MORSE_O
        LJMP MORSE_O

        ; CHECK FOR P
        NOT_MORSE_O:
        CJNE R4, #'.' , NOT_MORSE_P
        CJNE R5, #'-' , NOT_MORSE_P
        CJNE R6, #'-' , NOT_MORSE_P
        CJNE R7, #'.' , NOT_MORSE_P
        LJMP MORSE_P

        ; CHECK FOR Q
        NOT_MORSE_P:
        CJNE R4, #'-' , NOT_MORSE_Q
        CJNE R5, #'-' , NOT_MORSE_Q
        CJNE R6, #'.' , NOT_MORSE_Q
        CJNE R7, #'-' , NOT_MORSE_Q
        LJMP MORSE_Q

        ; CHECK FOR R
        NOT_MORSE_Q:
        CJNE R4, #'.' , NOT_MORSE_R
        CJNE R5, #'-' , NOT_MORSE_R
        CJNE R6, #'.' , NOT_MORSE_R
        CJNE R7, #0 , NOT_MORSE_R
        LJMP MORSE_R

        ; CHECK FOR S
        NOT_MORSE_R:
        CJNE R4, #'.' , NOT_MORSE_S
        CJNE R5, #'.' , NOT_MORSE_S
        CJNE R6, #'.' , NOT_MORSE_S
        CJNE R7, #0 , NOT_MORSE_S
        LJMP MORSE_S

        ; CHECK FOR T
        NOT_MORSE_S:
        CJNE R4, #'-' , NOT_MORSE_T
        CJNE R5, #0 , NOT_MORSE_T
        CJNE R6, #0 , NOT_MORSE_T
        CJNE R7, #0 , NOT_MORSE_T
        LJMP MORSE_T

        ; CHECK FOR U
        NOT_MORSE_T:
        CJNE R4, #'.' , NOT_MORSE_U
        CJNE R5, #'.' , NOT_MORSE_U
        CJNE R6, #'-' , NOT_MORSE_U
        CJNE R7, #0 , NOT_MORSE_U
        LJMP MORSE_U

        ; CHECK FOR V
        NOT_MORSE_U:
        CJNE R4, #'.' , NOT_MORSE_V
        CJNE R5, #'.' , NOT_MORSE_V
        CJNE R6, #'.' , NOT_MORSE_V
        CJNE R7, #'-' , NOT_MORSE_V
        LJMP MORSE_V

        ; CHECK FOR W
        NOT_MORSE_V:
        CJNE R4, #'.' , NOT_MORSE_W
        CJNE R5, #'-' , NOT_MORSE_W
        CJNE R6, #'-' , NOT_MORSE_W
        CJNE R7, #0 , NOT_MORSE_W
        LJMP MORSE_W

        ; CHECK FOR X
        NOT_MORSE_W:
        CJNE R4, #'-' , NOT_MORSE_X
        CJNE R5, #'.' , NOT_MORSE_X
        CJNE R6, #'.' , NOT_MORSE_X
        CJNE R7, #'-' , NOT_MORSE_X
        LJMP MORSE_X

        ; CHECK FOR Y
        NOT_MORSE_X:
        CJNE R4, #'-' , NOT_MORSE_Y
        CJNE R5, #'.' , NOT_MORSE_Y
        CJNE R6, #'-' , NOT_MORSE_Y
        CJNE R7, #'-' , NOT_MORSE_Y
        LJMP MORSE_Y

        ; CHECK FOR Z
        NOT_MORSE_Y:
        CJNE R4, #'-' , NOT_MORSE_Z
        CJNE R5, #'-' , NOT_MORSE_Z
        CJNE R6, #'.' , NOT_MORSE_Z
        CJNE R7, #'.' , NOT_MORSE_Z
        LJMP MORSE_Z

        NOT_MORSE_Z:
        LJMP MODE5

        MORSE_A:
            MOV A, #'A' 
            LJMP MORSE_LED

        MORSE_B:
            MOV A, #'B'
            LJMP MORSE_LED

        MORSE_C:
            MOV A, #'C'
            LJMP MORSE_LED

        MORSE_D:
            MOV A, #'D'
            LJMP MORSE_LED

        MORSE_E:
            MOV A, #'E'
            LJMP MORSE_LED

        MORSE_F:
            MOV A, #'F'
            LJMP MORSE_LED

        MORSE_G:
            MOV A, #'G'
            LJMP MORSE_LED

        MORSE_H:
            MOV A, #'H'
            LJMP MORSE_LED

        MORSE_I:
            MOV A, #'I'     
            LJMP MORSE_LED            

        MORSE_J:
            MOV A, #'J'
            LJMP MORSE_LED

        MORSE_K:
            MOV A, #'K'
            LJMP MORSE_LED

        MORSE_L:
            MOV A, #'L'
            LJMP MORSE_LED

        MORSE_M:
            MOV A, #'M'
            LJMP MORSE_LED

        MORSE_N:
            MOV A, #'N'
            LJMP MORSE_LED

        MORSE_O:
            MOV A, #'O'
            LJMP MORSE_LED

        MORSE_P:
            MOV A, #'P'
            LJMP MORSE_LED

        MORSE_Q:
            MOV A, #'Q'
            LJMP MORSE_LED

        MORSE_R:
            MOV A, #'R'
            LJMP MORSE_LED

        MORSE_S:
            MOV A, #'S'
            LJMP MORSE_LED

        MORSE_T:
            MOV A, #'T'
            LJMP MORSE_LED

        MORSE_U:
            MOV A, #'U'
            LJMP MORSE_LED

        MORSE_V:
            MOV A, #'V'
            LJMP MORSE_LED

        MORSE_W:
            MOV A, #'W'
            LJMP MORSE_LED

        MORSE_X:
            MOV A, #'X'        
            LJMP MORSE_LED

        MORSE_Y:
            MOV A, #'Y'
            LJMP MORSE_LED

        MORSE_Z:
            MOV A, #'Z'
            LJMP MORSE_LED        
     
        MORSE_LED:

            MOV @R0,A
            INC R0
            CHECK_R4:
                CJNE R4, #'-' , R4_NOT_DASH
                LJMP R4_DASH
            
                R4_NOT_DASH:
                    CJNE R4, #'.' , R4_NOT_DOT
                    LJMP R4_DOT

                R4_NOT_DOT:
                    CJNE R4, #'0' , R4_NOT_ZERO
                    LJMP R4_ZERO

                R4_NOT_ZERO:
                    LJMP INPUT_NEW_MORSE

                R4_DASH:
                    LCALL DASH
                    LJMP CHECK_R5
                R4_DOT:
                    LCALL DOT
                    LJMP CHECK_R5
                R4_ZERO:
                    LJMP INPUT_NEW_MORSE
                

            CHECK_R5:              
                CJNE R5, #'-' , R5_NOT_DASH
                LJMP R5_DASH
            
                R5_NOT_DASH:
                    CJNE R5, #'.' , R5_NOT_DOT
                    LJMP R5_DOT

                R5_NOT_DOT:
                    CJNE R5, #'0' , R5_NOT_ZERO
                    LJMP R5_ZERO

                R5_NOT_ZERO:
                    LJMP INPUT_NEW_MORSE
           
                R5_DASH:
                    LCALL DASH
                    LJMP CHECK_R6
                R5_DOT:
                    LCALL DOT
                    LJMP CHECK_R6                   
                R5_ZERO:
                    LJMP INPUT_NEW_MORSE

            CHECK_R6:              
                CJNE R6, #'-' , R6_NOT_DASH
                LJMP R6_DASH
            
                R6_NOT_DASH:
                    CJNE R6, #'.' , R6_NOT_DOT
                    LJMP R6_DOT

                R6_NOT_DOT:
                    CJNE R6, #'0' , R6_NOT_ZERO
                    LJMP R6_ZERO

                R6_NOT_ZERO:
                    LJMP INPUT_NEW_MORSE
           
                R6_DASH:
                    LCALL DASH
                    LJMP CHECK_R7
                R6_DOT:
                    LCALL DOT
                    LJMP CHECK_R7
                R6_ZERO:
                    LJMP INPUT_NEW_MORSE

            CHECK_R7:              
                CJNE R7, #'-' , R7_NOT_DASH
                LJMP R7_DASH
            
                R7_NOT_DASH:
                    CJNE R7, #'.' , R7_NOT_DOT
                    LJMP R7_DOT

                R7_NOT_DOT:
                    CJNE R7, #'0' , R7_NOT_ZERO
                    LJMP R7_ZERO

                R7_NOT_ZERO:
                    LJMP INPUT_NEW_MORSE
           
                R7_DASH:
                    LCALL DASH
                    LJMP INPUT_NEW_MORSE
                R7_DOT:
                    LCALL DOT
                    LJMP INPUT_NEW_MORSE
                R7_ZERO:
                    LJMP INPUT_NEW_MORSE



DISPLAY_CHAR:
MOV @R0, #00H
MOV R0, #50H

LCALL DELAY
MOV A,#0C0H ;2ND LINE
ACALL COMMAND
ACALL DELAY   

LOOP100:
MOV A, @R0
JZ EXIT_MODE5
ACALL DISPLAY
ACALL DELAY
INC R0
SJMP LOOP100

EXIT_MODE5:
ACALL GET_IP
CJNE A, #'0', EXIT_MODE5
MOV 32H, #16
LCALL CLEAR_LCD
MOV 32H, #16
LCALL CLEAR_LCD
LJMP select_mode


mode6:
    MOV 32H, #16
    LCALL CLEAR_LCD


acall GET_IP
ACALL DISPLAY
ACALL DELAY

CLR C 
SUBB A, #'0'
MOV R5,A

acall GET_IP
ACALL DISPLAY
ACALL DELAY

CLR C
SUBB A, #'0'

MOV B, #10
MUL AB
ADD A, R5
mov r5,A

acall GET_IP
ACALL DISPLAY
ACALL DELAY
MOV R6,A

acall GET_IP
ACALL DISPLAY
ACALL DELAY

CLR C 
SUBB A, #'0'
MOV R7,A

acall GET_IP
ACALL DISPLAY
ACALL DELAY

CLR C
SUBB A, #'0'

MOV B, #10
MUL AB
ADD A, R7
mov r7,A

MOV A, #'='
ACALL DISPLAY
ACALL DELAY

MOV A, R6
CJNE A, #'+', NEXT10
MOV A, R5
ADD A, R7

ACALL DISPLAY_OP
LJMP END_MODE6

NEXT10:
CJNE A, #'-', NEXT11
CLR C
MOV A, R5
SUBB A, R7
ACALL DISPLAY_OP
LJMP END_MODE6

NEXT11:

CLR C
MOV A, R5
MOV B, R7
MUL AB
ACALL DISPLAY_OP
LJMP END_MODE6

END_MODE6:
ACALL GET_IP
CJNE A, #'0', END_MODE6
MOV 32H, #16
LCALL CLEAR_LCD
LJMP select_mode


DISPLAY_OP:
MOV B, #10
DIV AB         ; A = A / 10, B = A % 10 → B = units
MOV R0, B      ; Temporarily store units

MOV B, #10
DIV AB         ; A = A / 10, B = A % 10 → B = tens
PUSH B         ; Push tens

PUSH 0        ; Push units

ADD A, #'0'    ; A = hundreds
ACALL DISPLAY
ACALL DELAY

POP ACC          ; A = tens
ADD A, #'0'
ACALL DISPLAY
ACALL DELAY

POP ACC; A = units
ADD A, #'0'
ACALL DISPLAY
ACALL DELAY
RET


CLEAR_LCD:
    LCALL DELAY
    MOV A,#01H ;clear lcd
    LCALL COMMAND
    LCALL DELAY    
RET      

GET_IP:
        ACALL DELAY
        CLR A
        CLR RI ;get ready to receive data
        WAIT_INPUT: JNB RI, WAIT_INPUT ;wait for the INPUT_NEW to come in
        mov a, sbuf
RET

COMMAND: 
    LCALL READY
    MOV P0,A
    CLR RS ;control reg is selected, send command to LCD controller
    CLR RW ;write to LCD 
    SETB ENBL
    LCALL DELAY
    CLR ENBL ;H to L pulse
RET

DISPLAY:    
    LCALL READY
    MOV P0,A
    SETB RS ;data register is selected, RS=1 send data to LCD
    CLR RW ;write to LCD
    SETB ENBL
    LCALL DELAY
    CLR ENBL ;H to L pulse
    DJNZ 32H, SAME_LINE
    LJMP NEW_LINE

    NEW_LINE:    
        LCALL DELAY
        MOV A,#0C0H ;2ND LINE
        LCALL COMMAND
        LCALL DELAY            
    SAME_LINE:
RET

;COMMAND and DISPLAY subroutine is same except the RS pin

READY:
    SETB P0.7
    CLR RS ;control reg is selected, send command to LCD controller
    SETB RW ;reading from LCD

WAIT:
    CLR ENBL ; disables communication with the LCD temporarily
    ACALL DELAY ;L to H pulse
    SETB ENBL
    JB P0.7,WAIT ;checks the busy flag
RET

DASH:            
    SETB LED_1

    LCALL DELAY
    LCALL DELAY
    LCALL DELAY
    LCALL DELAY
    LCALL DELAY
    LCALL DELAY            
    CLR LED_1
    LCALL DELAY
RET
            
DOT:
    SETB LED_1

    LCALL DELAY
    LCALL DELAY
    CLR LED_1

    LCALL DELAY
RET

DELAY:
    ; Save the current PSW
    push PSW            ; Save PSW in A (accumulator)
    ; Switch to Bank 2
    MOV PSW, #00010000B    ; RS1=1, RS0=0 (select Bank 2)
    MOV R1, #1            ; Using R1 from Bank 2
AGAIN_3:
    MOV R3, #220          ; Using R3 from Bank 2
AGAIN_2:
    MOV R4, #220          ; Using R4 from Bank 2 (delay)
AGAIN:
    DJNZ R4, AGAIN
    DJNZ R3, AGAIN_2
    DJNZ R1, AGAIN_3

    ; Restore the original PSW
    pop psw            ; Restore the saved PSW

RET

EXIT_2: SJMP EXIT_2

END
