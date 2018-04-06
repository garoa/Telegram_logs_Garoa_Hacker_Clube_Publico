;------------------------------------------
;   LEPUS 200
;------------------------------------------

eprom   equ     0x0000
ram     equ     0x4000

dat8251 equ     0xA000
cmd8251 equ     0xA001
sta8251 equ     0xA001

cmd8275 equ     0xD000
sta8275 equ     0xD000
par8275 equ     0xD000

pilha   equ     04026H

        org	00000H

;------------------------------------------
; RESET
;------------------------------------------
reset:
	di
	jmp	inicio

    ; ?
	db	000H, 004H, 0A3H, 09CH
    
    
identif:
	db	"LEPUS 200   Versao 2.", ' ' or 0x80
msgRam:
	db	"RA", 'M' or 0x80
msgEprom:
	db	"EPRO", M or 0x80
    
    ; filler
	db	000H, 000H, 000H, 000H
	db	000H, 000H


;------------------------------------------
;   Interrrupção da USART
;------------------------------------------
rst5:
	jmp	intSerial

    ; filler
	db	000H, 000H, 000H, 000H
	db	000H

;------------------------------------------
;   Interrrupção de Retraço Vertical
;------------------------------------------
rst6:
	jmp	intRetraco

    ; filler
	db	000H, 000H, 000H, 000H
	db	000H

;------------------------------------------
;   Interrrupção de Linha de Vídeo
;------------------------------------------
rst7:
	jmp	intLinVideo


;------------------------------------------
;   Iniciacao
;------------------------------------------
inicio:

    ; iniciação do video
        
	lxi	h,cmd8275
	mvi	m,000H      ; reset
	dcx	h
	mvi	m,04FH      ; 80 colunas
	mvi	m,059H      ; 1 linha de retraço, 26 linhas de vídeo
	mvi	m,079H      ; underline = linha 8,10 linhas por caracter
	mvi	m,06EH      ; 30 caracteres de retraço horizontal
                    ; cursor bloco não piscante
                    ; atributo visível
                    ; line counter mode 0
    
    ; Teste da Ram
    
	lxi	h,Ram       ; endereço inicial (0x4000)
	lxi	b,055AAH    ; padrões de teste
L0053:
	mov	m,b         ; escreve padrão atual
	inx	h           ; avança para a próxima posição
	mov	a,h
	cpi	048H
	jnz	L0053       ; repete até o fim da ram (0x47FF)

L005B:
	lxi	h,07850H
L005E:
	dcx	h
	mov	a,h
	ora	l
	jnz	L005E       ; dá um tempinho para ver se mantém
    
	lxi	h,Ram
L0067:
	mov	a,m
	cmp	b           ; compara
	jnz	L0081       ; escreveu, não leu -> erro
	mov	m,c         ; escreve o segundo padrão
	inx	h
	mov	a,h
	cpi	048H
	jnz	L0067       ; repete até o fim da Ram
    
	mov	a,c
	cpi	055H
	jz	L007F       ; fim se ja testou os dois padrões
	mov	c,b
	mov	b,a         
	jmp	L005B       ; confere o segundo padrão

    ; Teste da EProm
    
L007F:
	mvi	b,000H
L0081:
	lxi	h,01000H    ; fim da EProm
	xra	a           ; check = 0
L0085:
	dcx	h
	add	m
	mov	c,a
	mov	a,h
	ora	l
	mov	a,c
	jnz	L0085       ; soma todas as posiçoes da eprom
    
	lxi	sp,pilha    ; inicia a pilha
    
	push	b       ; salva resultados do teste (b= ram, c = eprom)
    
    ; iniciações dos módulos
    
	call	L00FC
	call	L0D2D
	call	L09FB
	call	L013B

    ; habilita interrupções
    
        mvi	a,018H      ; habilita ints 5, 6 e 7
	sim
        ei
    
	mvi	a,003H
	call	L0374
	mvi	a,00BH
	call	L0374
    
    ; Mostra resultado dos testes das memórias
	pop	b
	mov	a,b
	ana	a
	jz	L00BB
	lxi	h,msgRam
	lxi	d,04120H
	call	putMsg   ; avisa erro na Ram
L00BB:
	mov	a,c
	ana	a
	jz	L00C9
	lxi	h,msgEprom
	lxi	d,04118H
	call	putMsg   ; aviso erro no Eprom
        
    ; Se identifica
L00C9:
	lxi	h,identif
	lxi	d,043B3H
	call	putMsg
	mov	a,b
	ora	c
	jz	L00E3
