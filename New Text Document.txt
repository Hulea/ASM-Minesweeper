.386
.model flat, stdcall
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;

;includem biblioteci, si declaram ce functii vrem sa importam
includelib msvcrt.lib
extern exit: proc
extern malloc: proc
extern memset: proc
extern time:proc
extern rand:proc
extern srand:proc
extern printf:proc

includelib canvas.lib
extern BeginDrawing: proc
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;

;declaram simbolul start ca public - de acolo incepe executia
public start
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;

;sectiunile programului, date, respectiv cod
.data
;aici declaram date
window_title DB "Minesweeper ASM Hulea Andrei Grupa 5",0
area_width EQU 1080
area_height EQU 720
area DD 0

format db "%d ",0

counter DD 0 ; numara evenimentele de tip timer

matrice DD 100 dup(0);100 de elemente ale matricii initializate cu 0

x0 dd 100;colt stanga sus al tablei de joc desenate
y0 dd 100;-||-

x_matrice dd 0
y_matrice dd 0

var dd 0

coloana dd 0
linie dd 0

x dd 0
y dd 0

nr dd 0

i dd 0
j dd 0
imatr dd 0
jmatr dd 0

endl db " ",13,10,0
spatiu db " ",13,10,0

patrat_width equ 40;dimensiune patrat din tabel
patrat_height equ 40

verifica_afara DD 0

arg1 EQU 8
arg2 EQU 12
arg3 EQU 16
arg4 EQU 20

symbol_width EQU 10
symbol_height EQU 20
include digits.inc
include letters.inc

.code
; procedura make_text afiseaza o litera sau o cifra la coordonatele date
; arg1 - simbolul de afisat (litera sau cifra)
; arg2 - pointer la vectorul de pixeli
; arg3 - pos_x
; arg4 - pos_y
make_text proc
	push ebp
	mov ebp, esp
	pusha
	
	mov eax, [ebp+arg1] ; citim simbolul de afisat
	cmp eax, 'A'
	jl make_digit
	cmp eax, 'Z'
	jg make_digit
	sub eax, 'A'
	lea esi, letters
	jmp draw_text
make_digit:
	cmp eax, '0'
	jl make_space
	cmp eax, '9'
	jg make_space
	sub eax, '0'
	lea esi, digits
	jmp draw_text
make_space:	
	mov eax, 26 ; de la 0 pana la 25 sunt litere, 26 e space
	lea esi, letters
	
draw_text:
	mov ebx, symbol_width
	mul ebx
	mov ebx, symbol_height
	mul ebx
	add esi, eax
	mov ecx, symbol_height
bucla_simbol_linii:
	mov edi, [ebp+arg2] ; pointer la matricea de pixeli
	mov eax, [ebp+arg4] ; pointer la coord y
	add eax, symbol_height
	sub eax, ecx
	mov ebx, area_width
	mul ebx
	add eax, [ebp+arg3] ; pointer la coord x
	shl eax, 2 ; inmultim cu 4, avem un DWORD per pixel
	add edi, eax
	push ecx
	mov ecx, symbol_width
bucla_simbol_coloane:
	cmp byte ptr [esi], 0
	je simbol_pixel_alb
	mov dword ptr [edi], 0 ;CULOARE SCRIS
	jmp simbol_pixel_next
simbol_pixel_alb:
	mov dword ptr [edi], 0FFFFFFh; FUNDAL BACKGROUND TEXT
simbol_pixel_next:
	inc esi
	add edi, 4
	loop bucla_simbol_coloane
	pop ecx
	loop bucla_simbol_linii
	popa
	mov esp, ebp
	pop ebp
	ret
make_text endp

; un macro ca sa apelam mai usor desenarea simbolului
make_text_macro macro symbol, drawArea, x, y
	push y
	push x
	push drawArea
	push symbol
	call make_text
	add esp, 16
endm



deseneaza_linie_verticala macro x, y
local afis1, stop1
	pusha
	mov eax, x
	mov edx, area_width
	mul edx
	add eax, y
	shl eax, 2
	mov ecx, 0
afis1:
	mov ebx, [area]
	add eax, area_width*4
	mov dword ptr[ebx + eax],0;00FFFFh
	inc ecx
	cmp ecx, 400
	je stop1
	jmp afis1
stop1:
	popa
ENDM


deseneaza_linie_orizontala macro x, y
local afis2, stop2
	pusha
	mov eax, x
	mov ebx, area_width
	mul ebx
	add eax, y
	shl eax, 2
	mov ecx, 0
afis2:
	mov ebx, [area]
	add ebx, ecx
	mov dword ptr [ebx+eax],0; 0696969h
	inc ecx
	inc ecx
	cmp ecx, 1600
	je stop2
	loop afis2
stop2:
	popa
endm

deseneaza_linie_patratel_albastru macro x, y
local afis3, stop3
	pusha
	mov eax, x
	mov edx, area_width
	mul edx
	add eax, y
	shl eax, 2
	mov ecx, 0
afis3:
	mov ebx, [area]
	add eax, area_width*4
	mov dword ptr[ebx + eax],01e90ffh
	inc ecx
	cmp ecx, 37
	je stop3
	jmp afis3
stop3:
	popa
ENDM

deseneaza_linie_patratel_rosu macro x, y
local afis3, stop3
	pusha
	mov eax, x
	mov edx, area_width
	mul edx
	add eax, y
	shl eax, 2
	mov ecx, 0
afis3:
	mov ebx, [area]
	add eax, area_width*4
	mov dword ptr[ebx + eax],0ff0000fh
	inc ecx
	cmp ecx, 37
	je stop3
	jmp afis3
stop3:
	popa
ENDM

deseneaza_linie_patratel_gri macro x, y
local afis3, stop3
	pusha
	mov eax, x
	mov edx, area_width
	mul edx
	add eax, y
	shl eax, 2
	mov ecx, 0
afis3:
	mov ebx, [area]
	add eax, area_width*4
	mov dword ptr[ebx + eax],0C0C0C0h
	inc ecx
	cmp ecx, 37
	je stop3
	jmp afis3
stop3:
	popa
ENDM


deseneaza_patratel_rosu macro x,y
local patratel1
	mov ecx,37
	mov ebx,y
	mov eax,x
	patratel1:
		deseneaza_linie_patratel_rosu eax,ebx
		inc ebx
	loop patratel1
endm deseneaza_patratel_rosu

deseneaza_patratel_albastru macro x,y
local patratel2
	mov ecx,37
	mov ebx,y
	mov eax,x
	patratel2:
		deseneaza_linie_patratel_albastru eax,ebx
		inc ebx
	loop patratel2
endm deseneaza_patratel_albastru

deseneaza_patratel_gri macro x,y
local patratel3
	mov ecx,37
	mov ebx,y
	mov eax,x
	patratel3:
		deseneaza_linie_patratel_gri eax,ebx
		inc ebx
	loop patratel3
endm deseneaza_patratel_gri

; functia de desenare - se apeleaza la fiecare click
; sau la fiecare interval de 200ms in care nu s-a dat click
; arg1 - evt (0 - initializare, 1 - click, 2 - s-a scurs intervalul fara click)
; arg2 - x
; arg3 - y

draw proc
	push ebp
	mov ebp, esp
	pusha
	
	mov eax, [ebp+arg1]
	cmp eax, 1
	jz evt_click
	cmp eax, 2
	jz evt_timer ; nu s-a efectuat click pe nimic
	;mai jos e codul care intializeaza fereastra cu pixeli albi
	mov eax, area_width
	mov ebx, area_height
	mul ebx
	shl eax, 2
	push eax
	push 255
	push area
	call memset
	add esp, 12
	
	;linie,coloana
	
	deseneaza_linie_orizontala 100, 100
	deseneaza_linie_orizontala 140, 100
	deseneaza_linie_orizontala 180, 100
	deseneaza_linie_orizontala 220, 100
	deseneaza_linie_orizontala 260, 100
	deseneaza_linie_orizontala 300, 100
	deseneaza_linie_orizontala 340, 100
	deseneaza_linie_orizontala 380, 100
	deseneaza_linie_orizontala 420, 100
	deseneaza_linie_orizontala 460, 100
	deseneaza_linie_orizontala 500, 100

	
	deseneaza_linie_verticala 100, 100
	deseneaza_linie_verticala 100, 140
	deseneaza_linie_verticala 100, 180
	deseneaza_linie_verticala 100, 220
	deseneaza_linie_verticala 100, 260
	deseneaza_linie_verticala 100, 300
	deseneaza_linie_verticala 100, 340
	deseneaza_linie_verticala 100, 380
	deseneaza_linie_verticala 100, 420
	deseneaza_linie_verticala 100, 460
	deseneaza_linie_verticala 100, 500
	
	deseneaza_patratel_gri 101,102
	deseneaza_patratel_gri 141,102
	deseneaza_patratel_gri 181,102
	deseneaza_patratel_gri 221,102
	deseneaza_patratel_gri 261,102
	deseneaza_patratel_gri 301,102
	deseneaza_patratel_gri 341,102
	deseneaza_patratel_gri 381,102
	deseneaza_patratel_gri 421,102
	deseneaza_patratel_gri 461,102
	
	deseneaza_patratel_gri 101,142
	deseneaza_patratel_gri 141,142
	deseneaza_patratel_gri 181,142
	deseneaza_patratel_gri 221,142
	deseneaza_patratel_gri 261,142
	deseneaza_patratel_gri 301,142
	deseneaza_patratel_gri 341,142
	deseneaza_patratel_gri 381,142
	deseneaza_patratel_gri 421,142
	deseneaza_patratel_gri 461,142
	
	deseneaza_patratel_gri 101,182
	deseneaza_patratel_gri 141,182
	deseneaza_patratel_gri 181,182
	deseneaza_patratel_gri 221,182
	deseneaza_patratel_gri 261,182
	deseneaza_patratel_gri 301,182
	deseneaza_patratel_gri 341,182
	deseneaza_patratel_gri 381,182
	deseneaza_patratel_gri 421,182
	deseneaza_patratel_gri 461,182
	
	deseneaza_patratel_gri 101,222
	deseneaza_patratel_gri 141,222
	deseneaza_patratel_gri 181,222
	deseneaza_patratel_gri 221,222
	deseneaza_patratel_gri 261,222
	deseneaza_patratel_gri 301,222
	deseneaza_patratel_gri 341,222
	deseneaza_patratel_gri 381,222
	deseneaza_patratel_gri 421,222
	deseneaza_patratel_gri 461,222
	
	deseneaza_patratel_gri 101,262
	deseneaza_patratel_gri 141,262
	deseneaza_patratel_gri 181,262
	deseneaza_patratel_gri 221,262
	deseneaza_patratel_gri 261,262
	deseneaza_patratel_gri 301,262
	deseneaza_patratel_gri 341,262
	deseneaza_patratel_gri 381,262
	deseneaza_patratel_gri 421,262
	deseneaza_patratel_gri 461,262
	
	deseneaza_patratel_gri 101,302
	deseneaza_patratel_gri 141,302
	deseneaza_patratel_gri 181,302
	deseneaza_patratel_gri 221,302
	deseneaza_patratel_gri 261,302
	deseneaza_patratel_gri 301,302
	deseneaza_patratel_gri 341,302
	deseneaza_patratel_gri 381,302
	deseneaza_patratel_gri 421,302
	deseneaza_patratel_gri 461,302
	
	deseneaza_patratel_gri 101,342
	deseneaza_patratel_gri 141,342
	deseneaza_patratel_gri 181,342
	deseneaza_patratel_gri 221,342
	deseneaza_patratel_gri 261,342
	deseneaza_patratel_gri 301,342
	deseneaza_patratel_gri 341,342
	deseneaza_patratel_gri 381,342
	deseneaza_patratel_gri 421,342
	deseneaza_patratel_gri 461,342
	
	deseneaza_patratel_gri 101,382
	deseneaza_patratel_gri 141,382
	deseneaza_patratel_gri 181,382
	deseneaza_patratel_gri 221,382
	deseneaza_patratel_gri 261,382
	deseneaza_patratel_gri 301,382
	deseneaza_patratel_gri 341,382
	deseneaza_patratel_gri 381,382
	deseneaza_patratel_gri 421,382
	deseneaza_patratel_gri 461,382
	
	deseneaza_patratel_gri 101,422
	deseneaza_patratel_gri 141,422
	deseneaza_patratel_gri 181,422
	deseneaza_patratel_gri 221,422
	deseneaza_patratel_gri 261,422
	deseneaza_patratel_gri 301,422
	deseneaza_patratel_gri 341,422
	deseneaza_patratel_gri 381,422
	deseneaza_patratel_gri 421,422
	deseneaza_patratel_gri 461,422
	
	deseneaza_patratel_gri 101,462
	deseneaza_patratel_gri 141,462
	deseneaza_patratel_gri 181,462
	deseneaza_patratel_gri 221,462
	deseneaza_patratel_gri 261,462
	deseneaza_patratel_gri 301,462
	deseneaza_patratel_gri 341,462
	deseneaza_patratel_gri 381,462
	deseneaza_patratel_gri 421,462
	deseneaza_patratel_gri 461,462

	
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
	
	
	FORMARE_MATRICE_CU_BOMBE proc
	pusha
		xor ebx,ebx
		push ebx
		call time
		add esp,4
		push eax
		call srand
		add esp,4
	popa
	
	mov i,0
	again:
		pusha
			call rand
			xor ebx,ebx
			xor edx,edx
			mov ebx,10
			div ebx
			mov x,edx
			
			
		popa
		pusha
			call rand
			xor ebx,ebx
			xor edx,edx
			mov ebx,10
			div ebx
			mov y,edx
		popa
		
		xor ebx,ebx
		mov edx,0
		mov eax,x
		mov ebx,10
		mul ebx
		add eax,y
		shl eax, 2
		add eax,offset matrice
		mov dword ptr[eax],9; ELEMENTUL RANDOM, pe care l-am modificat din digits sa semene cu o bomba
		inc i
		cmp i, 10 ;NUMARUL DE ELEMENTE RANDOM DIN MATRICE
		jne again

	mov eax,0
	for3:
		cmp matrice[eax],9
		je e_bomba
		stanga_sus:
			cmp eax,0
			je dreapta
			cmp eax,4
			je stanga
			cmp eax,8
			je stanga
			cmp eax,12
			je stanga
			cmp eax,16
			je stanga
			cmp eax,20
			je stanga
			cmp eax,24
			je stanga
			cmp eax,28
			je stanga
			cmp eax,32
			je stanga
			cmp eax,36
			je stanga
			cmp eax,40
			je mijloc_sus
			cmp eax,80
			je mijloc_sus
			cmp eax,120
			je mijloc_sus
			cmp eax,160
			je mijloc_sus
			cmp eax,200
			je mijloc_sus
			cmp eax,240
			je mijloc_sus
			cmp eax,280
			je mijloc_sus
			cmp eax,320
			je mijloc_sus
			cmp eax,360
			je mijloc_sus
			cmp matrice[eax-44],9
			jne mijloc_sus
			add matrice[eax],1
		mijloc_sus:
			cmp matrice[eax-40],9
			jne dreapta_sus
			inc matrice[eax]
		dreapta_sus:
			cmp eax,76
			je stanga_jos
			cmp eax,116
			je stanga_jos
			cmp eax,156
			je stanga_jos
			cmp eax,196
			je stanga_jos
			cmp eax,236
			je stanga_jos
			cmp eax,276
			je stanga_jos
			cmp eax,316
			je stanga_jos
			cmp eax,356
			je stanga_jos
			cmp matrice[eax-36],9
			jne stanga
			inc matrice[eax]
		stanga:
			cmp eax,360
			je dreapta
			cmp matrice[eax-4],9
			jne dreapta
			inc matrice[eax]
		dreapta:
			cmp eax,36
			je stanga_jos
			cmp eax,76
			je stanga_jos
			cmp eax,116
			je stanga_jos
			cmp eax,156
			je stanga_jos
			cmp eax,196
			je stanga_jos
			cmp eax,236
			je stanga_jos
			cmp eax,276
			je stanga_jos
			cmp eax,316
			je stanga_jos
			cmp eax,356
			je stanga_jos
			cmp eax,396
			je stanga_jos
			cmp matrice[eax+4],9
			jne stanga_jos
			inc matrice[eax]
		stanga_jos:
			cmp eax,40
			je mijloc_jos
			cmp eax,80
			je mijloc_jos
			cmp eax,120
			je mijloc_jos
			cmp eax,160
			je mijloc_jos
			cmp eax,200
			je mijloc_jos
			cmp eax,240
			je mijloc_jos
			cmp eax,280
			je mijloc_jos
			cmp eax,320
			je mijloc_jos
			cmp eax,360
			je e_bomba
			cmp eax,364
			je e_bomba
			cmp eax,368
			je e_bomba
			cmp eax,372
			je e_bomba
			cmp eax,376
			je e_bomba
			cmp eax,380
			je e_bomba
			cmp eax,384
			je e_bomba
			cmp eax,388
			je e_bomba
			cmp eax,392
			je e_bomba
			cmp matrice[eax+36],9
			jne mijloc_jos
			inc matrice[eax]
		mijloc_jos:
			cmp matrice[eax+40],9
			jne dreapta_jos
			inc matrice[eax]
		dreapta_jos:
			cmp eax,36
			je e_bomba
			cmp eax,76
			je e_bomba
			cmp eax,116
			je e_bomba
			cmp eax,156
			je e_bomba
			cmp eax,196
			je e_bomba
			cmp eax,236
			je e_bomba
			cmp eax,276
			je e_bomba
			cmp eax,316
			je e_bomba
			cmp eax,356
			je e_bomba
			cmp matrice[eax+44],9
			jne e_bomba
			inc matrice[eax]
		
	e_bomba:
		add eax,4
		cmp eax,400
		jne for3
	
	mov eax,0
		
FORMARE_MATRICE_CU_BOMBE endp
	
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
		
	jmp afisare_litere

evt_click:	
;;;;;;;;;;;;
	mov edi,area
	mov ecx,[ebp+arg2]
	mov ebx,[ebp+arg3]
	
AFISARE_VALOARE:
	
	mov nr,0
	
LINIA_1: 
	cmp ebx,100
	jl afara
	cmp ebx,140
	jg LINIA_2
		COLOANA_1_LINIA_1: 
			cmp ecx,100
			jl afara
			cmp ecx,140
			jg COLOANA_2_LINIA_1
				cmp matrice[0],9
				jne alb11
				deseneaza_patratel_rosu 101,102
					make_text_macro 'G', area, 800, 200
					make_text_macro 'A', area, 810, 200
					make_text_macro 'M', area, 820, 200
					make_text_macro 'E', area, 830, 200

					make_text_macro 'O', area, 850, 200
					make_text_macro 'V', area, 860, 200
					make_text_macro 'E', area, 870, 200
					make_text_macro 'R', area, 880, 200
					call FORMARE_MATRICE_CU_BOMBE
				jmp aici11
				alb11:
				deseneaza_patratel_albastru 101,102
				inc nr
				aici11:
				add matrice[0],'0'
				make_text_macro matrice[0],area,115,110
				jmp afara
		COLOANA_2_LINIA_1:
			cmp ecx,180
			jg COLOANA_3_LINIA_1
			cmp ecx,140
			jl afara
				cmp matrice[4],9
				jne alb12
				deseneaza_patratel_rosu 101,142
					make_text_macro 'G', area, 800, 200
					make_text_macro 'A', area, 810, 200
					make_text_macro 'M', area, 820, 200
					make_text_macro 'E', area, 830, 200

					make_text_macro 'O', area, 850, 200
					make_text_macro 'V', area, 860, 200
					make_text_macro 'E', area, 870, 200
					make_text_macro 'R', area, 880, 200
				jmp aici12
				alb12:
				deseneaza_patratel_albastru 101,142
				inc nr
				aici12:
				add matrice[4],'0'
				make_text_macro matrice[4],area,155,110
		COLOANA_3_LINIA_1:
			cmp ecx,220
			jg COLOANA_4_LINIA_1
			cmp ecx,180
			jl afara
				cmp matrice[8],9
				jne alb13
				deseneaza_patratel_rosu 101,182
					make_text_macro 'G', area, 800, 200
					make_text_macro 'A', area, 810, 200
					make_text_macro 'M', area, 820, 200
					make_text_macro 'E', area, 830, 200

					make_text_macro 'O', area, 850, 200
					make_text_macro 'V', area, 860, 200
					make_text_macro 'E', area, 870, 200
					make_text_macro 'R', area, 880, 200
				jmp aici13
				alb13:
				deseneaza_patratel_albastru 101,182
				inc nr
				aici13:
				add matrice[8],'0'
				make_text_macro matrice[8],area,195,110
		COLOANA_4_LINIA_1:
			cmp ecx,260
			jg COLOANA_5_LINIA_1
			cmp ecx,220
			jl afara
				cmp matrice[12],9
				jne alb14
				deseneaza_patratel_rosu 101,222
					make_text_macro 'G', area, 800, 200
					make_text_macro 'A', area, 810, 200
					make_text_macro 'M', area, 820, 200
					make_text_macro 'E', area, 830, 200

					make_text_macro 'O', area, 850, 200
					make_text_macro 'V', area, 860, 200
					make_text_macro 'E', area, 870, 200
					make_text_macro 'R', area, 880, 200
				jmp aici14
				alb14:
				deseneaza_patratel_albastru 101,222
				inc nr
				aici14:
				add matrice[12],'0'
				make_text_macro matrice[12],area,235,110
		COLOANA_5_LINIA_1:
			cmp ecx,300
			jg COLOANA_6_LINIA_1
			cmp ecx,260
			jl afara
				cmp matrice[16],9
				jne alb15
				deseneaza_patratel_rosu 101,262
					make_text_macro 'G', area, 800, 200
					make_text_macro 'A', area, 810, 200
					make_text_macro 'M', area, 820, 200
					make_text_macro 'E', area, 830, 200

					make_text_macro 'O', area, 850, 200
					make_text_macro 'V', area, 860, 200
					make_text_macro 'E', area, 870, 200
					make_text_macro 'R', area, 880, 200
				jmp aici15
				alb15:
				deseneaza_patratel_albastru 101,262
				inc nr
				aici15:
				add matrice[16],'0'
				make_text_macro matrice[16],area,275,110
		COLOANA_6_LINIA_1:
			cmp ecx,340
			jg COLOANA_7_LINIA_1
			cmp ecx,300
			jl afara
				cmp matrice[20],9
				jne alb16
				deseneaza_patratel_rosu 101,302
					make_text_macro 'G', area, 800, 200
					make_text_macro 'A', area, 810, 200
					make_text_macro 'M', area, 820, 200
					make_text_macro 'E', area, 830, 200

					make_text_macro 'O', area, 850, 200
					make_text_macro 'V', area, 860, 200
					make_text_macro 'E', area, 870, 200
					make_text_macro 'R', area, 880, 200
				jmp aici16
				alb16:
				deseneaza_patratel_albastru 101,302
				inc nr
				aici16:
				add matrice[20],'0'
				make_text_macro matrice[20],area,315,110
		COLOANA_7_LINIA_1:
			cmp ecx,380
			jg COLOANA_8_LINIA_1
			cmp ecx,340
			jl afara
				cmp matrice[24],9
				jne alb17
				deseneaza_patratel_rosu 101,342
					make_text_macro 'G', area, 800, 200
					make_text_macro 'A', area, 810, 200
					make_text_macro 'M', area, 820, 200
					make_text_macro 'E', area, 830, 200

					make_text_macro 'O', area, 850, 200
					make_text_macro 'V', area, 860, 200
					make_text_macro 'E', area, 870, 200
					make_text_macro 'R', area, 880, 200
				jmp aici17
				alb17:
				deseneaza_patratel_albastru 101,342
				inc nr
				aici17:
				add matrice[24],'0'
				make_text_macro matrice[24],area,355,110
		COLOANA_8_LINIA_1:
			cmp ecx,420
			jg COLOANA_9_LINIA_1
			cmp ecx,380
			jl afara
				cmp matrice[28],9
				jne alb18
				deseneaza_patratel_rosu 101,382
					make_text_macro 'G', area, 800, 200
					make_text_macro 'A', area, 810, 200
					make_text_macro 'M', area, 820, 200
					make_text_macro 'E', area, 830, 200

					make_text_macro 'O', area, 850, 200
					make_text_macro 'V', area, 860, 200
					make_text_macro 'E', area, 870, 200
					make_text_macro 'R', area, 880, 200
				jmp aici18
				alb18:
				deseneaza_patratel_albastru 101,382
				inc nr
				aici18:
				add matrice[28],'0'
				make_text_macro matrice[28],area,395,110
		COLOANA_9_LINIA_1:
			cmp ecx,460
			jg COLOANA_10_LINIA_1
			cmp ecx,420
			jl afara
				cmp matrice[32],9
				jne alb19
				deseneaza_patratel_rosu 101,422
					make_text_macro 'G', area, 800, 200
					make_text_macro 'A', area, 810, 200
					make_text_macro 'M', area, 820, 200
					make_text_macro 'E', area, 830, 200

					make_text_macro 'O', area, 850, 200
					make_text_macro 'V', area, 860, 200
					make_text_macro 'E', area, 870, 200
					make_text_macro 'R', area, 880, 200
				jmp aici19
				alb19:
				deseneaza_patratel_albastru 101,422
				inc nr
				aici19:
				add matrice[32],'0'
				make_text_macro matrice[32],area,435,110
		COLOANA_10_LINIA_1:
			cmp ecx,500
			jg afara
			cmp ecx,460
			jl afara
				cmp matrice[36],9
				jne alb110
				deseneaza_patratel_rosu 101,462
					make_text_macro 'G', area, 800, 200
					make_text_macro 'A', area, 810, 200
					make_text_macro 'M', area, 820, 200
					make_text_macro 'E', area, 830, 200

					make_text_macro 'O', area, 850, 200
					make_text_macro 'V', area, 860, 200
					make_text_macro 'E', area, 870, 200
					make_text_macro 'R', area, 880, 200
				jmp aici110
				alb110:
				deseneaza_patratel_albastru 101,462
				inc nr
				aici110:
				add matrice[36],'0'
				make_text_macro matrice[36],area,475,110
		
LINIA_2:
	cmp ebx,140
	jl afara
	cmp ebx,180
	jg LINIA_3
		COLOANA_1_LINIA_2:
			cmp ecx,100
			jl afara
			cmp ecx,140
			jg COLOANA_2_LINIA_2
				cmp matrice[40],9
				jne alb21
				deseneaza_patratel_rosu 141,102
					make_text_macro 'G', area, 800, 200
					make_text_macro 'A', area, 810, 200
					make_text_macro 'M', area, 820, 200
					make_text_macro 'E', area, 830, 200

					make_text_macro 'O', area, 850, 200
					make_text_macro 'V', area, 860, 200
					make_text_macro 'E', area, 870, 200
					make_text_macro 'R', area, 880, 200
				jmp aici21
				alb21:
				deseneaza_patratel_albastru 141,102
				inc nr
				aici21:
				add matrice[40],'0'
				make_text_macro matrice[40],area,115,150
				jmp afara
		COLOANA_2_LINIA_2:
			cmp ecx,180
			jg COLOANA_3_LINIA_2
			cmp ecx,140
			jl afara
				cmp matrice[44],9
				jne alb22
				deseneaza_patratel_rosu 141,142
					make_text_macro 'G', area, 800, 200
					make_text_macro 'A', area, 810, 200
					make_text_macro 'M', area, 820, 200
					make_text_macro 'E', area, 830, 200

					make_text_macro 'O', area, 850, 200
					make_text_macro 'V', area, 860, 200
					make_text_macro 'E', area, 870, 200
					make_text_macro 'R', area, 880, 200
				jmp aici22
				alb22:
				deseneaza_patratel_albastru 141,142
				inc nr
				aici22:
				add matrice[44],'0'
				make_text_macro matrice[44],area,155,150
		COLOANA_3_LINIA_2:
			cmp ecx,220
			jg COLOANA_4_LINIA_2
			cmp ecx,180
			jl afara
				cmp matrice[48],9
				jne alb23
				deseneaza_patratel_rosu 141,182
					make_text_macro 'G', area, 800, 200
					make_text_macro 'A', area, 810, 200
					make_text_macro 'M', area, 820, 200
					make_text_macro 'E', area, 830, 200

					make_text_macro 'O', area, 850, 200
					make_text_macro 'V', area, 860, 200
					make_text_macro 'E', area, 870, 200
					make_text_macro 'R', area, 880, 200
				jmp aici23
				alb23:
				deseneaza_patratel_albastru 141,182
				inc nr
				aici23:
				add matrice[48],'0'
				make_text_macro matrice[48],area,195,150
		COLOANA_4_LINIA_2:
			cmp ecx,260
			jg COLOANA_5_LINIA_2
			cmp ecx,220
			jl afara
				cmp matrice[52],9
				jne alb24
				deseneaza_patratel_rosu 141,222
					make_text_macro 'G', area, 800, 200
					make_text_macro 'A', area, 810, 200
					make_text_macro 'M', area, 820, 200
					make_text_macro 'E', area, 830, 200

					make_text_macro 'O', area, 850, 200
					make_text_macro 'V', area, 860, 200
					make_text_macro 'E', area, 870, 200
					make_text_macro 'R', area, 880, 200
				jmp aici24
				alb24:
				deseneaza_patratel_albastru 141,222
				inc nr
				aici24:
				add matrice[52],'0'
				make_text_macro matrice[52],area,235,150
		COLOANA_5_LINIA_2:
			cmp ecx,300
			jg COLOANA_6_LINIA_2
			cmp ecx,260
			jl afara
				cmp matrice[56],9
				jne alb25
				deseneaza_patratel_rosu 141,262
					make_text_macro 'G', area, 800, 200
					make_text_macro 'A', area, 810, 200
					make_text_macro 'M', area, 820, 200
					make_text_macro 'E', area, 830, 200

					make_text_macro 'O', area, 850, 200
					make_text_macro 'V', area, 860, 200
					make_text_macro 'E', area, 870, 200
					make_text_macro 'R', area, 880, 200
				jmp aici25
				alb25:
				deseneaza_patratel_albastru 141,262
				inc nr
				aici25:
				add matrice[56],'0'
				make_text_macro matrice[56],area,275,150
		COLOANA_6_LINIA_2:
			cmp ecx,340
			jg COLOANA_7_LINIA_2
			cmp ecx,300
			jl afara
				cmp matrice[60],9
				jne alb26
				deseneaza_patratel_rosu 141,302
					make_text_macro 'G', area, 800, 200
					make_text_macro 'A', area, 810, 200
					make_text_macro 'M', area, 820, 200
					make_text_macro 'E', area, 830, 200

					make_text_macro 'O', area, 850, 200
					make_text_macro 'V', area, 860, 200
					make_text_macro 'E', area, 870, 200
					make_text_macro 'R', area, 880, 200
				jmp aici26
				alb26:
				deseneaza_patratel_albastru 141,302
				inc nr
				aici26:
				add matrice[60],'0'
				make_text_macro matrice[60],area,315,150
		COLOANA_7_LINIA_2:
			cmp ecx,380
			jg COLOANA_8_LINIA_2
			cmp ecx,340
			jl afara
				cmp matrice[64],9
				jne alb27
				deseneaza_patratel_rosu 141,342
					make_text_macro 'G', area, 800, 200
					make_text_macro 'A', area, 810, 200
					make_text_macro 'M', area, 820, 200
					make_text_macro 'E', area, 830, 200

					make_text_macro 'O', area, 850, 200
					make_text_macro 'V', area, 860, 200
					make_text_macro 'E', area, 870, 200
					make_text_macro 'R', area, 880, 200
				jmp aici27
				alb27:
				deseneaza_patratel_albastru 141,342
				inc nr
				aici27:
				add matrice[64],'0'
				make_text_macro matrice[64],area,355,150
		COLOANA_8_LINIA_2:
			cmp ecx,420
			jg COLOANA_9_LINIA_2
			cmp ecx,380
			jl afara
				cmp matrice[68],9
				jne alb28
				deseneaza_patratel_rosu 141,382
					make_text_macro 'G', area, 800, 200
					make_text_macro 'A', area, 810, 200
					make_text_macro 'M', area, 820, 200
					make_text_macro 'E', area, 830, 200

					make_text_macro 'O', area, 850, 200
					make_text_macro 'V', area, 860, 200
					make_text_macro 'E', area, 870, 200
					make_text_macro 'R', area, 880, 200
				jmp aici28
				alb28:
				deseneaza_patratel_albastru 141,382
				inc nr
				aici28:
				add matrice[68],'0'
				make_text_macro matrice[68],area,395,150
		COLOANA_9_LINIA_2:
			cmp ecx,460
			jg COLOANA_10_LINIA_2
			cmp ecx,420
			jl afara
				cmp matrice[72],9
				jne alb29
				deseneaza_patratel_rosu 141,422
					make_text_macro 'G', area, 800, 200
					make_text_macro 'A', area, 810, 200
					make_text_macro 'M', area, 820, 200
					make_text_macro 'E', area, 830, 200

					make_text_macro 'O', area, 850, 200
					make_text_macro 'V', area, 860, 200
					make_text_macro 'E', area, 870, 200
					make_text_macro 'R', area, 880, 200
				jmp aici29
				alb29:
				deseneaza_patratel_albastru 141,422
				inc nr
				aici29:
				add matrice[72],'0'
				make_text_macro matrice[72],area,435,150
		COLOANA_10_LINIA_2:
			cmp ecx,500
			jg afara
			cmp ecx,460
			jl afara
				cmp matrice[76],9
				jne alb210
				deseneaza_patratel_rosu 141,462
					make_text_macro 'G', area, 800, 200
					make_text_macro 'A', area, 810, 200
					make_text_macro 'M', area, 820, 200
					make_text_macro 'E', area, 830, 200

					make_text_macro 'O', area, 850, 200
					make_text_macro 'V', area, 860, 200
					make_text_macro 'E', area, 870, 200
					make_text_macro 'R', area, 880, 200
				jmp aici210
				alb210:
				deseneaza_patratel_albastru 141,462
				inc nr
				aici210:
				add matrice[76],'0'
				make_text_macro matrice[76],area,475,150

LINIA_3:
	cmp ebx,180
	jl afara
	cmp ebx,220
	jg LINIA_4
		COLOANA_1_LINIA_3:
			cmp ecx,100
			jl afara
			cmp ecx,140
			jg COLOANA_2_LINIA_3
				cmp matrice[80],9
				jne alb31
				deseneaza_patratel_rosu 181,102
					make_text_macro 'G', area, 800, 200
					make_text_macro 'A', area, 810, 200
					make_text_macro 'M', area, 820, 200
					make_text_macro 'E', area, 830, 200

					make_text_macro 'O', area, 850, 200
					make_text_macro 'V', area, 860, 200
					make_text_macro 'E', area, 870, 200
					make_text_macro 'R', area, 880, 200
				jmp aici31
				alb31:
				deseneaza_patratel_albastru 181,102
				inc nr
				aici31:
				add matrice[80],'0'
				make_text_macro matrice[80],area,115,190
				jmp afara
		COLOANA_2_LINIA_3:
			cmp ecx,180
			jg COLOANA_3_LINIA_3
			cmp ecx,140
			jl afara
				cmp matrice[84],9
				jne alb32
				deseneaza_patratel_rosu 181,142
					make_text_macro 'G', area, 800, 200
					make_text_macro 'A', area, 810, 200
					make_text_macro 'M', area, 820, 200
					make_text_macro 'E', area, 830, 200

					make_text_macro 'O', area, 850, 200
					make_text_macro 'V', area, 860, 200
					make_text_macro 'E', area, 870, 200
					make_text_macro 'R', area, 880, 200
				jmp aici32
				alb32:
				deseneaza_patratel_albastru 181,142
				inc nr
				aici32:
				add matrice[84],'0'
				make_text_macro matrice[84],area,155,190
		COLOANA_3_LINIA_3:
			cmp ecx,220
			jg COLOANA_4_LINIA_3
			cmp ecx,180
			jl afara
				cmp matrice[88],9
				jne alb33
				deseneaza_patratel_rosu 181,182
					make_text_macro 'G', area, 800, 200
					make_text_macro 'A', area, 810, 200
					make_text_macro 'M', area, 820, 200
					make_text_macro 'E', area, 830, 200

					make_text_macro 'O', area, 850, 200
					make_text_macro 'V', area, 860, 200
					make_text_macro 'E', area, 870, 200
					make_text_macro 'R', area, 880, 200
				jmp aici33
				alb33:
				deseneaza_patratel_albastru 181,182
				inc nr
				aici33:
				add matrice[88],'0'
				make_text_macro matrice[88],area,195,190
		COLOANA_4_LINIA_3:
			cmp ecx,260
			jg COLOANA_5_LINIA_3
			cmp ecx,220
			jl afara
				cmp matrice[92],9
				jne alb34
				deseneaza_patratel_rosu 181,222
					make_text_macro 'G', area, 800, 200
					make_text_macro 'A', area, 810, 200
					make_text_macro 'M', area, 820, 200
					make_text_macro 'E', area, 830, 200

					make_text_macro 'O', area, 850, 200
					make_text_macro 'V', area, 860, 200
					make_text_macro 'E', area, 870, 200
					make_text_macro 'R', area, 880, 200
				jmp aici34
				alb34:
				deseneaza_patratel_albastru 181,222
				inc nr
				aici34:
				add matrice[92],'0'
				make_text_macro matrice[92],area,235,190
		COLOANA_5_LINIA_3:
			cmp ecx,300
			jg COLOANA_6_LINIA_3
			cmp ecx,260
			jl afara
				cmp matrice[96],9
				jne alb35
				deseneaza_patratel_rosu 181,262
					make_text_macro 'G', area, 800, 200
					make_text_macro 'A', area, 810, 200
					make_text_macro 'M', area, 820, 200
					make_text_macro 'E', area, 830, 200

					make_text_macro 'O', area, 850, 200
					make_text_macro 'V', area, 860, 200
					make_text_macro 'E', area, 870, 200
					make_text_macro 'R', area, 880, 200
				jmp aici35
				alb35:
				deseneaza_patratel_albastru 181,262
				inc nr
				aici35:
				add matrice[96],'0'
				make_text_macro matrice[96],area,275,190
		COLOANA_6_LINIA_3:
			cmp ecx,340
			jg COLOANA_7_LINIA_3
			cmp ecx,300
			jl afara
				cmp matrice[100],9
				jne alb36
				deseneaza_patratel_rosu 181,302
					make_text_macro 'G', area, 800, 200
					make_text_macro 'A', area, 810, 200
					make_text_macro 'M', area, 820, 200
					make_text_macro 'E', area, 830, 200

					make_text_macro 'O', area, 850, 200
					make_text_macro 'V', area, 860, 200
					make_text_macro 'E', area, 870, 200
					make_text_macro 'R', area, 880, 200
				jmp aici36
				alb36:
				deseneaza_patratel_albastru 181,302
				inc nr
				aici36:
				add matrice[100],'0'
				make_text_macro matrice[100],area,315,190
		COLOANA_7_LINIA_3:
			cmp ecx,380
			jg COLOANA_8_LINIA_3
			cmp ecx,340
			jl afara
				cmp matrice[104],9
				jne alb37
				deseneaza_patratel_rosu 181,342
					make_text_macro 'G', area, 800, 200
					make_text_macro 'A', area, 810, 200
					make_text_macro 'M', area, 820, 200
					make_text_macro 'E', area, 830, 200

					make_text_macro 'O', area, 850, 200
					make_text_macro 'V', area, 860, 200
					make_text_macro 'E', area, 870, 200
					make_text_macro 'R', area, 880, 200
				jmp aici37
				alb37:
				deseneaza_patratel_albastru 181,342
				inc nr
				aici37:
				add matrice[104],'0'
				make_text_macro matrice[104],area,355,190
		COLOANA_8_LINIA_3:
			cmp ecx,420
			jg COLOANA_9_LINIA_3
			cmp ecx,380
			jl afara
				cmp matrice[108],9
				jne alb38
				deseneaza_patratel_rosu 181,382
					make_text_macro 'G', area, 800, 200
					make_text_macro 'A', area, 810, 200
					make_text_macro 'M', area, 820, 200
					make_text_macro 'E', area, 830, 200

					make_text_macro 'O', area, 850, 200
					make_text_macro 'V', area, 860, 200
					make_text_macro 'E', area, 870, 200
					make_text_macro 'R', area, 880, 200
				jmp aici38
				alb38:
				deseneaza_patratel_albastru 181,382
				inc nr
				aici38:
				add matrice[108],'0'
				make_text_macro matrice[108],area,395,190
		COLOANA_9_LINIA_3:
			cmp ecx,460
			jg COLOANA_10_LINIA_3
			cmp ecx,420
			jl afara
				cmp matrice[112],9
				jne alb39
				deseneaza_patratel_rosu 181,422
					make_text_macro 'G', area, 800, 200
					make_text_macro 'A', area, 810, 200
					make_text_macro 'M', area, 820, 200
					make_text_macro 'E', area, 830, 200

					make_text_macro 'O', area, 850, 200
					make_text_macro 'V', area, 860, 200
					make_text_macro 'E', area, 870, 200
					make_text_macro 'R', area, 880, 200
				jmp aici39
				alb39:
				deseneaza_patratel_albastru 181,422
				inc nr
				aici39:
				add matrice[112],'0'
				make_text_macro matrice[112],area,435,190
		COLOANA_10_LINIA_3:
			cmp ecx,500
			jg afara
			cmp ecx,460
			jl afara
				cmp matrice[116],9
				jne alb310
				deseneaza_patratel_rosu 181,462
					make_text_macro 'G', area, 800, 200
					make_text_macro 'A', area, 810, 200
					make_text_macro 'M', area, 820, 200
					make_text_macro 'E', area, 830, 200

					make_text_macro 'O', area, 850, 200
					make_text_macro 'V', area, 860, 200
					make_text_macro 'E', area, 870, 200
					make_text_macro 'R', area, 880, 200
				jmp aici310
				alb310:
				deseneaza_patratel_albastru 181,462
				inc nr
				aici310:
				add matrice[116],'0'
				make_text_macro matrice[116],area,475,190	
LINIA_4:
	cmp ebx,220
	jl afara
	cmp ebx,260
	jg LINIA_5
		COLOANA_1_LINIA_4:
			cmp ecx,100
			jl afara
			cmp ecx,140
			jg COLOANA_2_LINIA_4
				cmp matrice[120],9
				jne alb41
				deseneaza_patratel_rosu 221,102
					make_text_macro 'G', area, 800, 200
					make_text_macro 'A', area, 810, 200
					make_text_macro 'M', area, 820, 200
					make_text_macro 'E', area, 830, 200

					make_text_macro 'O', area, 850, 200
					make_text_macro 'V', area, 860, 200
					make_text_macro 'E', area, 870, 200
					make_text_macro 'R', area, 880, 200
				jmp aici41
				alb41:
				deseneaza_patratel_albastru 221,102
				inc nr
				aici41:
				add matrice[120],'0'
				make_text_macro matrice[120],area,115,230
				jmp afara
		COLOANA_2_LINIA_4:
			cmp ecx,180
			jg COLOANA_3_LINIA_4
			cmp ecx,140
			jl afara
				cmp matrice[124],9
				jne alb42
				deseneaza_patratel_rosu 221,142
					make_text_macro 'G', area, 800, 200
					make_text_macro 'A', area, 810, 200
					make_text_macro 'M', area, 820, 200
					make_text_macro 'E', area, 830, 200

					make_text_macro 'O', area, 850, 200
					make_text_macro 'V', area, 860, 200
					make_text_macro 'E', area, 870, 200
					make_text_macro 'R', area, 880, 200
				jmp aici42
				alb42:
				deseneaza_patratel_albastru 221,142
				inc nr
				aici42:
				add matrice[124],'0'
				make_text_macro matrice[124],area,155,230
		COLOANA_3_LINIA_4:
			cmp ecx,220
			jg COLOANA_4_LINIA_4
			cmp ecx,180
			jl afara
				cmp matrice[128],9
				jne alb43
				deseneaza_patratel_rosu 221,182
					make_text_macro 'G', area, 800, 200
					make_text_macro 'A', area, 810, 200
					make_text_macro 'M', area, 820, 200
					make_text_macro 'E', area, 830, 200

					make_text_macro 'O', area, 850, 200
					make_text_macro 'V', area, 860, 200
					make_text_macro 'E', area, 870, 200
					make_text_macro 'R', area, 880, 200
				jmp aici43
				alb43:
				deseneaza_patratel_albastru 221,182
				inc nr
				aici43:
				add matrice[128],'0'
				make_text_macro matrice[128],area,195,230
		COLOANA_4_LINIA_4:
			cmp ecx,260
			jg COLOANA_5_LINIA_4
			cmp ecx,220
			jl afara
				cmp matrice[132],9
				jne alb44
				deseneaza_patratel_rosu 221,222
					make_text_macro 'G', area, 800, 200
					make_text_macro 'A', area, 810, 200
					make_text_macro 'M', area, 820, 200
					make_text_macro 'E', area, 830, 200

					make_text_macro 'O', area, 850, 200
					make_text_macro 'V', area, 860, 200
					make_text_macro 'E', area, 870, 200
					make_text_macro 'R', area, 880, 200
				jmp aici44
				alb44:
				deseneaza_patratel_albastru 221,222
				inc nr
				aici44:
				add matrice[132],'0'
				make_text_macro matrice[132],area,235,230
		COLOANA_5_LINIA_4:
			cmp ecx,300
			jg COLOANA_6_LINIA_4
			cmp ecx,260
			jl afara
				cmp matrice[136],9
				jne alb45
				deseneaza_patratel_rosu 221,262
					make_text_macro 'G', area, 800, 200
					make_text_macro 'A', area, 810, 200
					make_text_macro 'M', area, 820, 200
					make_text_macro 'E', area, 830, 200

					make_text_macro 'O', area, 850, 200
					make_text_macro 'V', area, 860, 200
					make_text_macro 'E', area, 870, 200
					make_text_macro 'R', area, 880, 200
				jmp aici45
				alb45:
				deseneaza_patratel_albastru 221,262
				inc nr
				aici45:
				add matrice[136],'0'
				make_text_macro matrice[136],area,275,230
		COLOANA_6_LINIA_4:
			cmp ecx,340
			jg COLOANA_7_LINIA_4
			cmp ecx,300
			jl afara
				cmp matrice[140],9
				jne alb46
				deseneaza_patratel_rosu 221,302
					make_text_macro 'G', area, 800, 200
					make_text_macro 'A', area, 810, 200
					make_text_macro 'M', area, 820, 200
					make_text_macro 'E', area, 830, 200

					make_text_macro 'O', area, 850, 200
					make_text_macro 'V', area, 860, 200
					make_text_macro 'E', area, 870, 200
					make_text_macro 'R', area, 880, 200
				jmp aici46
				alb46:
				deseneaza_patratel_albastru 221,302
				inc nr
				aici46:
				add matrice[140],'0'
				make_text_macro matrice[140],area,315,230
		COLOANA_7_LINIA_4:
			cmp ecx,380
			jg COLOANA_8_LINIA_4
			cmp ecx,340
			jl afara
				cmp matrice[144],9
				jne alb47
				deseneaza_patratel_rosu 221,342
					make_text_macro 'G', area, 800, 200
					make_text_macro 'A', area, 810, 200
					make_text_macro 'M', area, 820, 200
					make_text_macro 'E', area, 830, 200

					make_text_macro 'O', area, 850, 200
					make_text_macro 'V', area, 860, 200
					make_text_macro 'E', area, 870, 200
					make_text_macro 'R', area, 880, 200
				jmp aici47
				alb47:
				deseneaza_patratel_albastru 221,342
				inc nr
				aici47:
				add matrice[144],'0'
				make_text_macro matrice[144],area,355,230
		COLOANA_8_LINIA_4:
			cmp ecx,420
			jg COLOANA_9_LINIA_4
			cmp ecx,380
			jl afara
				cmp matrice[148],9
				jne alb48
				deseneaza_patratel_rosu 221,382
					make_text_macro 'G', area, 800, 200
					make_text_macro 'A', area, 810, 200
					make_text_macro 'M', area, 820, 200
					make_text_macro 'E', area, 830, 200

					make_text_macro 'O', area, 850, 200
					make_text_macro 'V', area, 860, 200
					make_text_macro 'E', area, 870, 200
					make_text_macro 'R', area, 880, 200
				jmp aici48
				alb48:
				deseneaza_patratel_albastru 221,382
				inc nr
				aici48:
				add matrice[148],'0'
				make_text_macro matrice[148],area,395,230
		COLOANA_9_LINIA_4:
			cmp ecx,460
			jg COLOANA_10_LINIA_4
			cmp ecx,420
			jl afara
				cmp matrice[152],9
				jne alb49
				deseneaza_patratel_rosu 221,422
					make_text_macro 'G', area, 800, 200
					make_text_macro 'A', area, 810, 200
					make_text_macro 'M', area, 820, 200
					make_text_macro 'E', area, 830, 200

					make_text_macro 'O', area, 850, 200
					make_text_macro 'V', area, 860, 200
					make_text_macro 'E', area, 870, 200
					make_text_macro 'R', area, 880, 200
				jmp aici49
				alb49:
				deseneaza_patratel_albastru 221,422
				inc nr
				aici49:
				add matrice[152],'0'
				make_text_macro matrice[152],area,435,230
		COLOANA_10_LINIA_4:
			cmp ecx,500
			jg afara
			cmp ecx,460
			jl afara
				cmp matrice[156],9
				jne alb410
				deseneaza_patratel_rosu 221,462
					make_text_macro 'G', area, 800, 200
					make_text_macro 'A', area, 810, 200
					make_text_macro 'M', area, 820, 200
					make_text_macro 'E', area, 830, 200

					make_text_macro 'O', area, 850, 200
					make_text_macro 'V', area, 860, 200
					make_text_macro 'E', area, 870, 200
					make_text_macro 'R', area, 880, 200
				jmp aici410
				alb410:
				deseneaza_patratel_albastru 221,462
				inc nr
				aici410:
				add matrice[156],'0'
				make_text_macro matrice[156],area,475,230
LINIA_5:
	cmp ebx,260
	jl afara
	cmp ebx,300
	jg LINIA_6
		COLOANA_1_LINIA_5:
			cmp ecx,100
			jl afara
			cmp ecx,140
			jg COLOANA_2_LINIA_5
				cmp matrice[160],9
				jne alb51
				deseneaza_patratel_rosu 261,102
					make_text_macro 'G', area, 800, 200
					make_text_macro 'A', area, 810, 200
					make_text_macro 'M', area, 820, 200
					make_text_macro 'E', area, 830, 200

					make_text_macro 'O', area, 850, 200
					make_text_macro 'V', area, 860, 200
					make_text_macro 'E', area, 870, 200
					make_text_macro 'R', area, 880, 200
				jmp aici51
				alb51:
				deseneaza_patratel_albastru 261,102
				inc nr
				aici51:
				add matrice[160],'0'
				make_text_macro matrice[160],area,115,270
				jmp afara
		COLOANA_2_LINIA_5:
			cmp ecx,180
			jg COLOANA_3_LINIA_5
			cmp ecx,140
			jl afara
			cmp matrice[164],9
				jne alb52
				deseneaza_patratel_rosu 261,142
					make_text_macro 'G', area, 800, 200
					make_text_macro 'A', area, 810, 200
					make_text_macro 'M', area, 820, 200
					make_text_macro 'E', area, 830, 200

					make_text_macro 'O', area, 850, 200
					make_text_macro 'V', area, 860, 200
					make_text_macro 'E', area, 870, 200
					make_text_macro 'R', area, 880, 200
				jmp aici52
				alb52:
				deseneaza_patratel_albastru 261,142
				inc nr
				aici52:
				add matrice[164],'0'
				make_text_macro matrice[164],area,155,270
		COLOANA_3_LINIA_5:
			cmp ecx,220
			jg COLOANA_4_LINIA_5
			cmp ecx,180
			jl afara
				cmp matrice[168],9
				jne alb53
				deseneaza_patratel_rosu 261,182
					make_text_macro 'G', area, 800, 200
					make_text_macro 'A', area, 810, 200
					make_text_macro 'M', area, 820, 200
					make_text_macro 'E', area, 830, 200

					make_text_macro 'O', area, 850, 200
					make_text_macro 'V', area, 860, 200
					make_text_macro 'E', area, 870, 200
					make_text_macro 'R', area, 880, 200
				jmp aici53
				alb53:
				deseneaza_patratel_albastru 261,182
				inc nr
				aici53:
				add matrice[168],'0'
				make_text_macro matrice[168],area,195,270
		COLOANA_4_LINIA_5:
			cmp ecx,260
			jg COLOANA_5_LINIA_5
			cmp ecx,220
			jl afara
				cmp matrice[172],9
				jne alb54
				deseneaza_patratel_rosu 261,222
					make_text_macro 'G', area, 800, 200
					make_text_macro 'A', area, 810, 200
					make_text_macro 'M', area, 820, 200
					make_text_macro 'E', area, 830, 200

					make_text_macro 'O', area, 850, 200
					make_text_macro 'V', area, 860, 200
					make_text_macro 'E', area, 870, 200
					make_text_macro 'R', area, 880, 200
				jmp aici54
				alb54:
				deseneaza_patratel_albastru 261,222
				inc nr
				aici54:
				add matrice[172],'0'
				make_text_macro matrice[172],area,235,270
		COLOANA_5_LINIA_5:
			cmp ecx,300
			jg COLOANA_6_LINIA_5
			cmp ecx,260
			jl afara
				cmp matrice[176],9
				jne alb55
				deseneaza_patratel_rosu 261,262
					make_text_macro 'G', area, 800, 200
					make_text_macro 'A', area, 810, 200
					make_text_macro 'M', area, 820, 200
					make_text_macro 'E', area, 830, 200

					make_text_macro 'O', area, 850, 200
					make_text_macro 'V', area, 860, 200
					make_text_macro 'E', area, 870, 200
					make_text_macro 'R', area, 880, 200
				jmp aici55
				alb55:
				deseneaza_patratel_albastru 261,262
				inc nr
				aici55:
				add matrice[176],'0'
				make_text_macro matrice[176],area,275,270
		COLOANA_6_LINIA_5:
			cmp ecx,340
			jg COLOANA_7_LINIA_5
			cmp ecx,300
			jl afara
				cmp matrice[180],9
				jne alb56
				deseneaza_patratel_rosu 261,302
					make_text_macro 'G', area, 800, 200
					make_text_macro 'A', area, 810, 200
					make_text_macro 'M', area, 820, 200
					make_text_macro 'E', area, 830, 200

					make_text_macro 'O', area, 850, 200
					make_text_macro 'V', area, 860, 200
					make_text_macro 'E', area, 870, 200
					make_text_macro 'R', area, 880, 200
				jmp aici56
				alb56:
				deseneaza_patratel_albastru 261,302
				inc nr
				aici56:
				add matrice[180],'0'
				make_text_macro matrice[180],area,315,270
		COLOANA_7_LINIA_5:
			cmp ecx,380
			jg COLOANA_8_LINIA_5
			cmp ecx,340
			jl afara
			cmp matrice[184],9
				jne alb57
				deseneaza_patratel_rosu 261,342
					make_text_macro 'G', area, 800, 200
					make_text_macro 'A', area, 810, 200
					make_text_macro 'M', area, 820, 200
					make_text_macro 'E', area, 830, 200

					make_text_macro 'O', area, 850, 200
					make_text_macro 'V', area, 860, 200
					make_text_macro 'E', area, 870, 200
					make_text_macro 'R', area, 880, 200
				jmp aici57
				alb57:
				deseneaza_patratel_albastru 261,342
				inc nr
				aici57:
				add matrice[184],'0'
				make_text_macro matrice[184],area,355,270
		COLOANA_8_LINIA_5:
			cmp ecx,420
			jg COLOANA_9_LINIA_5
			cmp ecx,380
			jl afara
				cmp matrice[188],9
				jne alb58
				deseneaza_patratel_rosu 261,382
					make_text_macro 'G', area, 800, 200
					make_text_macro 'A', area, 810, 200
					make_text_macro 'M', area, 820, 200
					make_text_macro 'E', area, 830, 200

					make_text_macro 'O', area, 850, 200
					make_text_macro 'V', area, 860, 200
					make_text_macro 'E', area, 870, 200
					make_text_macro 'R', area, 880, 200
				jmp aici58
				alb58:
				deseneaza_patratel_albastru 261,382
				inc nr
				aici58:
				add matrice[188],'0'
				make_text_macro matrice[188],area,395,270
		COLOANA_9_LINIA_5:
			cmp ecx,460
			jg COLOANA_10_LINIA_5
			cmp ecx,420
			jl afara
				cmp matrice[192],9
				jne alb59
				deseneaza_patratel_rosu 261,422
					make_text_macro 'G', area, 800, 200
					make_text_macro 'A', area, 810, 200
					make_text_macro 'M', area, 820, 200
					make_text_macro 'E', area, 830, 200

					make_text_macro 'O', area, 850, 200
					make_text_macro 'V', area, 860, 200
					make_text_macro 'E', area, 870, 200
					make_text_macro 'R', area, 880, 200
				jmp aici59
				alb59:
				deseneaza_patratel_albastru 261,422
				inc nr
				aici59:
				add matrice[192],'0'
				make_text_macro matrice[192],area,435,270
		COLOANA_10_LINIA_5:
			cmp ecx,500
			jg afara
			cmp ecx,460
			jl afara
				cmp matrice[196],9
				jne alb510
				deseneaza_patratel_rosu 261,462
					make_text_macro 'G', area, 800, 200
					make_text_macro 'A', area, 810, 200
					make_text_macro 'M', area, 820, 200
					make_text_macro 'E', area, 830, 200

					make_text_macro 'O', area, 850, 200
					make_text_macro 'V', area, 860, 200
					make_text_macro 'E', area, 870, 200
					make_text_macro 'R', area, 880, 200
				jmp aici510
				alb510:
				deseneaza_patratel_albastru 261,462
				inc nr
				aici510:
				add matrice[196],'0'
				make_text_macro matrice[196],area,475,270
LINIA_6:
	cmp ebx,300
	jl afara
	cmp ebx,340
	jg LINIA_7
		COLOANA_1_LINIA_6:
			cmp ecx,100
			jl afara
			cmp ecx,140
			jg COLOANA_2_LINIA_6
				cmp matrice[200],9
				jne alb61
				deseneaza_patratel_rosu 301,102
					make_text_macro 'G', area, 800, 200
					make_text_macro 'A', area, 810, 200
					make_text_macro 'M', area, 820, 200
					make_text_macro 'E', area, 830, 200

					make_text_macro 'O', area, 850, 200
					make_text_macro 'V', area, 860, 200
					make_text_macro 'E', area, 870, 200
					make_text_macro 'R', area, 880, 200
				jmp aici61
				alb61:
				deseneaza_patratel_albastru 301,102
				inc nr
				aici61:
				add matrice[200],'0'
				make_text_macro matrice[200],area,115,310
		COLOANA_2_LINIA_6:
			cmp ecx,180
			jg COLOANA_3_LINIA_6
			cmp ecx,140
			jl afara
				cmp matrice[204],9
				jne alb62
				deseneaza_patratel_rosu 301,142
					make_text_macro 'G', area, 800, 200
					make_text_macro 'A', area, 810, 200
					make_text_macro 'M', area, 820, 200
					make_text_macro 'E', area, 830, 200

					make_text_macro 'O', area, 850, 200
					make_text_macro 'V', area, 860, 200
					make_text_macro 'E', area, 870, 200
					make_text_macro 'R', area, 880, 200
				jmp aici62
				alb62:
				deseneaza_patratel_albastru 301,142
				inc nr
				aici62:
				add matrice[204],'0'
				make_text_macro matrice[204],area,155,310
		COLOANA_3_LINIA_6:
			cmp ecx,220
			jg COLOANA_4_LINIA_6
			cmp ecx,180
			jl afara
				cmp matrice[208],9
				jne alb63
				deseneaza_patratel_rosu 301,182
					make_text_macro 'G', area, 800, 200
					make_text_macro 'A', area, 810, 200
					make_text_macro 'M', area, 820, 200
					make_text_macro 'E', area, 830, 200

					make_text_macro 'O', area, 850, 200
					make_text_macro 'V', area, 860, 200
					make_text_macro 'E', area, 870, 200
					make_text_macro 'R', area, 880, 200
				jmp aici63
				alb63:
				deseneaza_patratel_albastru 301,182
				inc nr
				aici63:
				add matrice[208],'0'
				make_text_macro matrice[208],area,195,310
		COLOANA_4_LINIA_6:
			cmp ecx,260
			jg COLOANA_5_LINIA_6
			cmp ecx,220
			jl afara
				cmp matrice[212],9
				jne alb64
				deseneaza_patratel_rosu 301,222
					make_text_macro 'G', area, 800, 200
					make_text_macro 'A', area, 810, 200
					make_text_macro 'M', area, 820, 200
					make_text_macro 'E', area, 830, 200

					make_text_macro 'O', area, 850, 200
					make_text_macro 'V', area, 860, 200
					make_text_macro 'E', area, 870, 200
					make_text_macro 'R', area, 880, 200
				jmp aici64
				alb64:
				deseneaza_patratel_albastru 301,222
				inc nr
				aici64:
				add matrice[212],'0'
				make_text_macro matrice[212],area,235,310
		COLOANA_5_LINIA_6:
			cmp ecx,300
			jg COLOANA_6_LINIA_6
			cmp ecx,260
			jl afara
				cmp matrice[216],9
				jne alb65
				deseneaza_patratel_rosu 301,262
					make_text_macro 'G', area, 800, 200
					make_text_macro 'A', area, 810, 200
					make_text_macro 'M', area, 820, 200
					make_text_macro 'E', area, 830, 200

					make_text_macro 'O', area, 850, 200
					make_text_macro 'V', area, 860, 200
					make_text_macro 'E', area, 870, 200
					make_text_macro 'R', area, 880, 200
				jmp aici65
				alb65:
				deseneaza_patratel_albastru 301,262
				inc nr
				aici65:
				add matrice[216],'0'
				make_text_macro matrice[216],area,275,310
		COLOANA_6_LINIA_6:
			cmp ecx,340
			jg COLOANA_7_LINIA_6
			cmp ecx,300
			jl afara
				cmp matrice[220],9
				jne alb66
				deseneaza_patratel_rosu 301,302
					make_text_macro 'G', area, 800, 200
					make_text_macro 'A', area, 810, 200
					make_text_macro 'M', area, 820, 200
					make_text_macro 'E', area, 830, 200

					make_text_macro 'O', area, 850, 200
					make_text_macro 'V', area, 860, 200
					make_text_macro 'E', area, 870, 200
					make_text_macro 'R', area, 880, 200
				jmp aici66
				alb66:
				deseneaza_patratel_albastru 301,302
				inc nr
				aici66:
				add matrice[220],'0'
				make_text_macro matrice[220],area,315,310
		COLOANA_7_LINIA_6:
			cmp ecx,380
			jg COLOANA_8_LINIA_6
			cmp ecx,340
			jl afara
				cmp matrice[224],9
				jne alb67
				deseneaza_patratel_rosu 301,342
					make_text_macro 'G', area, 800, 200
					make_text_macro 'A', area, 810, 200
					make_text_macro 'M', area, 820, 200
					make_text_macro 'E', area, 830, 200

					make_text_macro 'O', area, 850, 200
					make_text_macro 'V', area, 860, 200
					make_text_macro 'E', area, 870, 200
					make_text_macro 'R', area, 880, 200
				jmp aici67
				alb67:
				deseneaza_patratel_albastru 301,342
				inc nr
				aici67:
				add matrice[224],'0'
				make_text_macro matrice[224],area,355,310
		COLOANA_8_LINIA_6:
			cmp ecx,420
			jg COLOANA_9_LINIA_6
			cmp ecx,380
			jl afara
				cmp matrice[228],9
				jne alb68
				deseneaza_patratel_rosu 301,382
					make_text_macro 'G', area, 800, 200
					make_text_macro 'A', area, 810, 200
					make_text_macro 'M', area, 820, 200
					make_text_macro 'E', area, 830, 200

					make_text_macro 'O', area, 850, 200
					make_text_macro 'V', area, 860, 200
					make_text_macro 'E', area, 870, 200
					make_text_macro 'R', area, 880, 200
				jmp aici68
				alb68:
				deseneaza_patratel_albastru 301,382
				inc nr
				aici68:
				add matrice[228],'0'
				make_text_macro matrice[228],area,395,310
		COLOANA_9_LINIA_6:
			cmp ecx,460
			jg COLOANA_10_LINIA_6
			cmp ecx,420
			jl afara
				cmp matrice[232],9
				jne alb69
				deseneaza_patratel_rosu 301,422
					make_text_macro 'G', area, 800, 200
					make_text_macro 'A', area, 810, 200
					make_text_macro 'M', area, 820, 200
					make_text_macro 'E', area, 830, 200

					make_text_macro 'O', area, 850, 200
					make_text_macro 'V', area, 860, 200
					make_text_macro 'E', area, 870, 200
					make_text_macro 'R', area, 880, 200
				jmp aici69
				alb69:
				deseneaza_patratel_albastru 301,422
				inc nr
				aici69:
				add matrice[232],'0'
				make_text_macro matrice[232],area,435,310
		COLOANA_10_LINIA_6:
			cmp ecx,500
			jg afara
			cmp ecx,460
			jl afara
				cmp matrice[236],9
				jne alb610
				deseneaza_patratel_rosu 301,462
					make_text_macro 'G', area, 800, 200
					make_text_macro 'A', area, 810, 200
					make_text_macro 'M', area, 820, 200
					make_text_macro 'E', area, 830, 200

					make_text_macro 'O', area, 850, 200
					make_text_macro 'V', area, 860, 200
					make_text_macro 'E', area, 870, 200
					make_text_macro 'R', area, 880, 200
				jmp aici610
				alb610:
				deseneaza_patratel_albastru 301,462
				inc nr
				aici610:
				add matrice[236],'0'
				make_text_macro matrice[236],area,475,310
LINIA_7:
	cmp ebx,340
	jl afara
	cmp ebx,380
	jg LINIA_8
		COLOANA_1_LINIA_7:
			cmp ecx,100
			jl afara
			cmp ecx,140
			jg COLOANA_2_LINIA_7
				cmp matrice[240],9
				jne alb71
				deseneaza_patratel_rosu 341,102
					make_text_macro 'G', area, 800, 200
					make_text_macro 'A', area, 810, 200
					make_text_macro 'M', area, 820, 200
					make_text_macro 'E', area, 830, 200

					make_text_macro 'O', area, 850, 200
					make_text_macro 'V', area, 860, 200
					make_text_macro 'E', area, 870, 200
					make_text_macro 'R', area, 880, 200
				jmp aici71
				alb71:
				deseneaza_patratel_albastru 341,102
				inc nr
				aici71:
				add matrice[240],'0'
				make_text_macro matrice[240],area,115,350
				jmp afara
		COLOANA_2_LINIA_7:
			cmp ecx,180
			jg COLOANA_3_LINIA_7
			cmp ecx,140
			jl afara
				cmp matrice[244],9
				jne alb72
				deseneaza_patratel_rosu 341,142
					make_text_macro 'G', area, 800, 200
					make_text_macro 'A', area, 810, 200
					make_text_macro 'M', area, 820, 200
					make_text_macro 'E', area, 830, 200

					make_text_macro 'O', area, 850, 200
					make_text_macro 'V', area, 860, 200
					make_text_macro 'E', area, 870, 200
					make_text_macro 'R', area, 880, 200
				jmp aici72
				alb72:
				deseneaza_patratel_albastru 341,142
				inc nr
				aici72:
				add matrice[244],'0'
				make_text_macro matrice[244],area,155,350
		COLOANA_3_LINIA_7:
			cmp ecx,220
			jg COLOANA_4_LINIA_7
			cmp ecx,180
			jl afara
				cmp matrice[248],9
				jne alb73
				deseneaza_patratel_rosu 341,182
					make_text_macro 'G', area, 800, 200
					make_text_macro 'A', area, 810, 200
					make_text_macro 'M', area, 820, 200
					make_text_macro 'E', area, 830, 200

					make_text_macro 'O', area, 850, 200
					make_text_macro 'V', area, 860, 200
					make_text_macro 'E', area, 870, 200
					make_text_macro 'R', area, 880, 200
				jmp aici73
				alb73:
				deseneaza_patratel_albastru 341,182
				inc nr
				aici73:
				add matrice[248],'0'
				make_text_macro matrice[248],area,195,350
		COLOANA_4_LINIA_7:
			cmp ecx,260
			jg COLOANA_5_LINIA_7
			cmp ecx,220
			jl afara
				cmp matrice[252],9
				jne alb74
				deseneaza_patratel_rosu 341,222
					make_text_macro 'G', area, 800, 200
					make_text_macro 'A', area, 810, 200
					make_text_macro 'M', area, 820, 200
					make_text_macro 'E', area, 830, 200

					make_text_macro 'O', area, 850, 200
					make_text_macro 'V', area, 860, 200
					make_text_macro 'E', area, 870, 200
					make_text_macro 'R', area, 880, 200
				jmp aici74
				alb74:
				deseneaza_patratel_albastru 341,222
				inc nr
				aici74:
				add matrice[252],'0'
				make_text_macro matrice[252],area,235,350
		COLOANA_5_LINIA_7:
			cmp ecx,300
			jg COLOANA_6_LINIA_7
			cmp ecx,260
			jl afara
				cmp matrice[256],9
				jne alb75
				deseneaza_patratel_rosu 341,262
					make_text_macro 'G', area, 800, 200
					make_text_macro 'A', area, 810, 200
					make_text_macro 'M', area, 820, 200
					make_text_macro 'E', area, 830, 200

					make_text_macro 'O', area, 850, 200
					make_text_macro 'V', area, 860, 200
					make_text_macro 'E', area, 870, 200
					make_text_macro 'R', area, 880, 200
				jmp aici75
				alb75:
				deseneaza_patratel_albastru 341,262
				inc nr
				aici75:
				add matrice[256],'0'
				make_text_macro matrice[256],area,275,350
		COLOANA_6_LINIA_7:
			cmp ecx,340
			jg COLOANA_7_LINIA_7
			cmp ecx,300
			jl afara
				cmp matrice[260],9
				jne alb76
				deseneaza_patratel_rosu 341,302
					make_text_macro 'G', area, 800, 200
					make_text_macro 'A', area, 810, 200
					make_text_macro 'M', area, 820, 200
					make_text_macro 'E', area, 830, 200

					make_text_macro 'O', area, 850, 200
					make_text_macro 'V', area, 860, 200
					make_text_macro 'E', area, 870, 200
					make_text_macro 'R', area, 880, 200
				jmp aici76
				alb76:
				deseneaza_patratel_albastru 341,302
				inc nr
				aici76:
				add matrice[260],'0'
				make_text_macro matrice[260],area,315,350
		COLOANA_7_LINIA_7:
			cmp ecx,380
			jg COLOANA_8_LINIA_7
			cmp ecx,340
			jl afara
				cmp matrice[264],9
				jne alb77
				deseneaza_patratel_rosu 341,342
					make_text_macro 'G', area, 800, 200
					make_text_macro 'A', area, 810, 200
					make_text_macro 'M', area, 820, 200
					make_text_macro 'E', area, 830, 200

					make_text_macro 'O', area, 850, 200
					make_text_macro 'V', area, 860, 200
					make_text_macro 'E', area, 870, 200
					make_text_macro 'R', area, 880, 200
				jmp aici77
				alb77:
				deseneaza_patratel_albastru 341,342
				inc nr
				aici77:
				add matrice[264],'0'
				make_text_macro matrice[264],area,355,350
		COLOANA_8_LINIA_7:
			cmp ecx,420
			jg COLOANA_9_LINIA_7
			cmp ecx,380
			jl afara
				cmp matrice[268],9
				jne alb78
				deseneaza_patratel_rosu 341,382
					make_text_macro 'G', area, 800, 200
					make_text_macro 'A', area, 810, 200
					make_text_macro 'M', area, 820, 200
					make_text_macro 'E', area, 830, 200

					make_text_macro 'O', area, 850, 200
					make_text_macro 'V', area, 860, 200
					make_text_macro 'E', area, 870, 200
					make_text_macro 'R', area, 880, 200
				jmp aici78
				alb78:
				deseneaza_patratel_albastru 341,382
				inc nr
				aici78:
				add matrice[268],'0'
				make_text_macro matrice[268],area,395,350
		COLOANA_9_LINIA_7:
			cmp ecx,460
			jg COLOANA_10_LINIA_7
			cmp ecx,420
			jl afara
				cmp matrice[272],9
				jne alb79
				deseneaza_patratel_rosu 341,422
					make_text_macro 'G', area, 800, 200
					make_text_macro 'A', area, 810, 200
					make_text_macro 'M', area, 820, 200
					make_text_macro 'E', area, 830, 200

					make_text_macro 'O', area, 850, 200
					make_text_macro 'V', area, 860, 200
					make_text_macro 'E', area, 870, 200
					make_text_macro 'R', area, 880, 200
				jmp aici79
				alb79:
				deseneaza_patratel_albastru 341,422
				inc nr
				aici79:
				add matrice[272],'0'
				make_text_macro matrice[272],area,435,350
		COLOANA_10_LINIA_7:
			cmp ecx,500
			jg afara
			cmp ecx,460
			jl afara
				cmp matrice[276],9
				jne alb710
				deseneaza_patratel_rosu 341,462
					make_text_macro 'G', area, 800, 200
					make_text_macro 'A', area, 810, 200
					make_text_macro 'M', area, 820, 200
					make_text_macro 'E', area, 830, 200

					make_text_macro 'O', area, 850, 200
					make_text_macro 'V', area, 860, 200
					make_text_macro 'E', area, 870, 200
					make_text_macro 'R', area, 880, 200
				jmp aici710
				alb710:
				deseneaza_patratel_albastru 341,462
				inc nr
				aici710:
				add matrice[276],'0'
				make_text_macro matrice[276],area,475,350
LINIA_8:
	cmp ebx,380
	jl afara
	cmp ebx,420
	jg LINIA_9
		COLOANA_1_LINIA_8:
			cmp ecx,100
			jl afara
			cmp ecx,140
			jg COLOANA_2_LINIA_8
				cmp matrice[280],9
				jne alb81
				deseneaza_patratel_rosu 381,102
					make_text_macro 'G', area, 800, 200
					make_text_macro 'A', area, 810, 200
					make_text_macro 'M', area, 820, 200
					make_text_macro 'E', area, 830, 200

					make_text_macro 'O', area, 850, 200
					make_text_macro 'V', area, 860, 200
					make_text_macro 'E', area, 870, 200
					make_text_macro 'R', area, 880, 200
				jmp aici81
				alb81:
				deseneaza_patratel_albastru 381,102
				inc nr
				aici81:
				add matrice[280],'0'
				make_text_macro matrice[280],area,115,390
				jmp afara
		COLOANA_2_LINIA_8:
			cmp ecx,180
			jg COLOANA_3_LINIA_8
			cmp ecx,140
			jl afara
				cmp matrice[284],9
				jne alb82
				deseneaza_patratel_rosu 381,142
					make_text_macro 'G', area, 800, 200
					make_text_macro 'A', area, 810, 200
					make_text_macro 'M', area, 820, 200
					make_text_macro 'E', area, 830, 200

					make_text_macro 'O', area, 850, 200
					make_text_macro 'V', area, 860, 200
					make_text_macro 'E', area, 870, 200
					make_text_macro 'R', area, 880, 200
				jmp aici82
				alb82:
				deseneaza_patratel_albastru 381,142
				inc nr
				aici82:
				add matrice[284],'0'
				make_text_macro matrice[284],area,155,390
		COLOANA_3_LINIA_8:
			cmp ecx,220
			jg COLOANA_4_LINIA_8
			cmp ecx,180
			jl afara
				cmp matrice[288],9
				jne alb83
				deseneaza_patratel_rosu 381,182
					make_text_macro 'G', area, 800, 200
					make_text_macro 'A', area, 810, 200
					make_text_macro 'M', area, 820, 200
					make_text_macro 'E', area, 830, 200

					make_text_macro 'O', area, 850, 200
					make_text_macro 'V', area, 860, 200
					make_text_macro 'E', area, 870, 200
					make_text_macro 'R', area, 880, 200
				jmp aici83
				alb83:
				deseneaza_patratel_albastru 381,182
				inc nr
				aici83:
				add matrice[288],'0'
				make_text_macro matrice[288],area,195,390
		COLOANA_4_LINIA_8:
			cmp ecx,260
			jg COLOANA_5_LINIA_8
			cmp ecx,220
			jl afara
				cmp matrice[292],9
				jne alb84
				deseneaza_patratel_rosu 381,222
					make_text_macro 'G', area, 800, 200
					make_text_macro 'A', area, 810, 200
					make_text_macro 'M', area, 820, 200
					make_text_macro 'E', area, 830, 200

					make_text_macro 'O', area, 850, 200
					make_text_macro 'V', area, 860, 200
					make_text_macro 'E', area, 870, 200
					make_text_macro 'R', area, 880, 200
				jmp aici84
				alb84:
				deseneaza_patratel_albastru 381,222
				inc nr
				aici84:
				add matrice[292],'0'
				make_text_macro matrice[292],area,235,390
		COLOANA_5_LINIA_8:
			cmp ecx,300
			jg COLOANA_6_LINIA_8
			cmp ecx,260
			jl afara
				cmp matrice[296],9
				jne alb85
				deseneaza_patratel_rosu 381,262
					make_text_macro 'G', area, 800, 200
					make_text_macro 'A', area, 810, 200
					make_text_macro 'M', area, 820, 200
					make_text_macro 'E', area, 830, 200

					make_text_macro 'O', area, 850, 200
					make_text_macro 'V', area, 860, 200
					make_text_macro 'E', area, 870, 200
					make_text_macro 'R', area, 880, 200
				jmp aici85
				alb85:
				deseneaza_patratel_albastru 381,262
				inc nr
				aici85:
				add matrice[296],'0'
				make_text_macro matrice[296],area,275,390
		COLOANA_6_LINIA_8:
			cmp ecx,340
			jg COLOANA_7_LINIA_8
			cmp ecx,300
			jl afara
				cmp matrice[300],9
				jne alb86
				deseneaza_patratel_rosu 381,302
					make_text_macro 'G', area, 800, 200
					make_text_macro 'A', area, 810, 200
					make_text_macro 'M', area, 820, 200
					make_text_macro 'E', area, 830, 200

					make_text_macro 'O', area, 850, 200
					make_text_macro 'V', area, 860, 200
					make_text_macro 'E', area, 870, 200
					make_text_macro 'R', area, 880, 200
				jmp aici86
				alb86:
				deseneaza_patratel_albastru 381,302
				inc nr
				aici86:
				add matrice[300],'0'
				make_text_macro matrice[300],area,315,390
		COLOANA_7_LINIA_8:
			cmp ecx,380
			jg COLOANA_8_LINIA_8
			cmp ecx,340
			jl afara
				cmp matrice[304],9
				jne alb87
				deseneaza_patratel_rosu 381,342
					make_text_macro 'G', area, 800, 200
					make_text_macro 'A', area, 810, 200
					make_text_macro 'M', area, 820, 200
					make_text_macro 'E', area, 830, 200

					make_text_macro 'O', area, 850, 200
					make_text_macro 'V', area, 860, 200
					make_text_macro 'E', area, 870, 200
					make_text_macro 'R', area, 880, 200
				jmp aici87
				alb87:
				deseneaza_patratel_albastru 381,342
				inc nr
				aici87:
				add matrice[304],'0'
				make_text_macro matrice[304],area,355,390
		COLOANA_8_LINIA_8:
			cmp ecx,420
			jg COLOANA_9_LINIA_8
			cmp ecx,380
			jl afara
				cmp matrice[308],9
				jne alb88
				deseneaza_patratel_rosu 381,382
					make_text_macro 'G', area, 800, 200
					make_text_macro 'A', area, 810, 200
					make_text_macro 'M', area, 820, 200
					make_text_macro 'E', area, 830, 200

					make_text_macro 'O', area, 850, 200
					make_text_macro 'V', area, 860, 200
					make_text_macro 'E', area, 870, 200
					make_text_macro 'R', area, 880, 200
				jmp aici88
				alb88:
				deseneaza_patratel_albastru 381,382
				inc nr
				aici88:
				add matrice[308],'0'
				make_text_macro matrice[308],area,395,390
		COLOANA_9_LINIA_8:
			cmp ecx,460
			jg COLOANA_10_LINIA_8
			cmp ecx,420
			jl afara
				cmp matrice[312],9
				jne alb89
				deseneaza_patratel_rosu 381,422
					make_text_macro 'G', area, 800, 200
					make_text_macro 'A', area, 810, 200
					make_text_macro 'M', area, 820, 200
					make_text_macro 'E', area, 830, 200

					make_text_macro 'O', area, 850, 200
					make_text_macro 'V', area, 860, 200
					make_text_macro 'E', area, 870, 200
					make_text_macro 'R', area, 880, 200
				jmp aici89
				alb89:
				deseneaza_patratel_albastru 381,422
				inc nr
				aici89:
				add matrice[312],'0'
				make_text_macro matrice[312],area,435,390
		COLOANA_10_LINIA_8:
			cmp ecx,500
			jg afara
			cmp ecx,460
			jl afara
				cmp matrice[316],9
				jne alb810
				deseneaza_patratel_rosu 381,462
					make_text_macro 'G', area, 800, 200
					make_text_macro 'A', area, 810, 200
					make_text_macro 'M', area, 820, 200
					make_text_macro 'E', area, 830, 200

					make_text_macro 'O', area, 850, 200
					make_text_macro 'V', area, 860, 200
					make_text_macro 'E', area, 870, 200
					make_text_macro 'R', area, 880, 200
				jmp aici810
				alb810:
				deseneaza_patratel_albastru 381,462
				inc nr
				aici810:
				add matrice[316],'0'
				make_text_macro matrice[316],area,475,390
LINIA_9:
	cmp ebx,420
	jl afara
	cmp ebx,460
	jg LINIA_10
		COLOANA_1_LINIA_9:
			cmp ecx,100
			jl afara
			cmp ecx,140
			jg COLOANA_2_LINIA_9
				cmp matrice[320],9
				jne alb91
				deseneaza_patratel_rosu 421,102
					make_text_macro 'G', area, 800, 200
					make_text_macro 'A', area, 810, 200
					make_text_macro 'M', area, 820, 200
					make_text_macro 'E', area, 830, 200

					make_text_macro 'O', area, 850, 200
					make_text_macro 'V', area, 860, 200
					make_text_macro 'E', area, 870, 200
					make_text_macro 'R', area, 880, 200
				jmp aici91
				alb91:
				deseneaza_patratel_albastru 421,102
				inc nr
				aici91:
				add matrice[320],'0'
				make_text_macro matrice[320],area,115,430
				jmp afara
		COLOANA_2_LINIA_9:
			cmp ecx,180
			jg COLOANA_3_LINIA_9
			cmp ecx,140
			jl afara
			cmp matrice[324],9
				jne alb92
				deseneaza_patratel_rosu 421,142
					make_text_macro 'G', area, 800, 200
					make_text_macro 'A', area, 810, 200
					make_text_macro 'M', area, 820, 200
					make_text_macro 'E', area, 830, 200

					make_text_macro 'O', area, 850, 200
					make_text_macro 'V', area, 860, 200
					make_text_macro 'E', area, 870, 200
					make_text_macro 'R', area, 880, 200
				jmp aici92
				alb92:
				deseneaza_patratel_albastru 421,142
				inc nr
				aici92:
				add matrice[324],'0'
				make_text_macro matrice[324],area,155,430
		COLOANA_3_LINIA_9:
			cmp ecx,220
			jg COLOANA_4_LINIA_9
			cmp ecx,180
			jl afara
				cmp matrice[328],9
				jne alb93
				deseneaza_patratel_rosu 421,182
					make_text_macro 'G', area, 800, 200
					make_text_macro 'A', area, 810, 200
					make_text_macro 'M', area, 820, 200
					make_text_macro 'E', area, 830, 200

					make_text_macro 'O', area, 850, 200
					make_text_macro 'V', area, 860, 200
					make_text_macro 'E', area, 870, 200
					make_text_macro 'R', area, 880, 200
				jmp aici93
				alb93:
				deseneaza_patratel_albastru 421,182
				inc nr
				aici93:
				add matrice[328],'0'
				make_text_macro matrice[328],area,195,430
		COLOANA_4_LINIA_9:
			cmp ecx,260
			jg COLOANA_5_LINIA_9
			cmp ecx,220
			jl afara
				cmp matrice[332],9
				jne alb94
				deseneaza_patratel_rosu 421,222
					make_text_macro 'G', area, 800, 200
					make_text_macro 'A', area, 810, 200
					make_text_macro 'M', area, 820, 200
					make_text_macro 'E', area, 830, 200

					make_text_macro 'O', area, 850, 200
					make_text_macro 'V', area, 860, 200
					make_text_macro 'E', area, 870, 200
					make_text_macro 'R', area, 880, 200
				jmp aici94
				alb94:
				deseneaza_patratel_albastru 421,222
				inc nr
				aici94:
				add matrice[332],'0'
				make_text_macro matrice[332],area,235,430
		COLOANA_5_LINIA_9:
			cmp ecx,300
			jg COLOANA_6_LINIA_9
			cmp ecx,260
			jl afara
				cmp matrice[336],9
				jne alb95
				deseneaza_patratel_rosu 421,262
					make_text_macro 'G', area, 800, 200
					make_text_macro 'A', area, 810, 200
					make_text_macro 'M', area, 820, 200
					make_text_macro 'E', area, 830, 200

					make_text_macro 'O', area, 850, 200
					make_text_macro 'V', area, 860, 200
					make_text_macro 'E', area, 870, 200
					make_text_macro 'R', area, 880, 200
				jmp aici95
				alb95:
				deseneaza_patratel_albastru 421,262
				inc nr
				aici95:
				add matrice[336],'0'
				make_text_macro matrice[336],area,275,430
		COLOANA_6_LINIA_9:
			cmp ecx,340
			jg COLOANA_7_LINIA_9
			cmp ecx,300
			jl afara
				cmp matrice[340],9
				jne alb96
				deseneaza_patratel_rosu 421,302
					make_text_macro 'G', area, 800, 200
					make_text_macro 'A', area, 810, 200
					make_text_macro 'M', area, 820, 200
					make_text_macro 'E', area, 830, 200

					make_text_macro 'O', area, 850, 200
					make_text_macro 'V', area, 860, 200
					make_text_macro 'E', area, 870, 200
					make_text_macro 'R', area, 880, 200
				jmp aici96
				alb96:
				deseneaza_patratel_albastru 421,302
				inc nr
				aici96:
				add matrice[340],'0'
				make_text_macro matrice[340],area,315,430
		COLOANA_7_LINIA_9:
			cmp ecx,380
			jg COLOANA_8_LINIA_9
			cmp ecx,340
			jl afara
				cmp matrice[344],9
				jne alb97
				deseneaza_patratel_rosu 421,342
					make_text_macro 'G', area, 800, 200
					make_text_macro 'A', area, 810, 200
					make_text_macro 'M', area, 820, 200
					make_text_macro 'E', area, 830, 200

					make_text_macro 'O', area, 850, 200
					make_text_macro 'V', area, 860, 200
					make_text_macro 'E', area, 870, 200
					make_text_macro 'R', area, 880, 200
				jmp aici97
				alb97:
				deseneaza_patratel_albastru 421,342
				inc nr
				aici97:
				add matrice[344],'0'
				make_text_macro matrice[344],area,355,430
		COLOANA_8_LINIA_9:
			cmp ecx,420
			jg COLOANA_9_LINIA_9
			cmp ecx,380
			jl afara
				cmp matrice[348],9
				jne alb98
				deseneaza_patratel_rosu 421,382
					make_text_macro 'G', area, 800, 200
					make_text_macro 'A', area, 810, 200
					make_text_macro 'M', area, 820, 200
					make_text_macro 'E', area, 830, 200

					make_text_macro 'O', area, 850, 200
					make_text_macro 'V', area, 860, 200
					make_text_macro 'E', area, 870, 200
					make_text_macro 'R', area, 880, 200
				jmp aici98
				alb98:
				deseneaza_patratel_albastru 421,382
				inc nr
				aici98:
				add matrice[348],'0'
				make_text_macro matrice[348],area,395,430
		COLOANA_9_LINIA_9:
			cmp ecx,460
			jg COLOANA_10_LINIA_9
			cmp ecx,420
			jl afara
				cmp matrice[352],9
				jne alb99
				deseneaza_patratel_rosu 421,422
					make_text_macro 'G', area, 800, 200
					make_text_macro 'A', area, 810, 200
					make_text_macro 'M', area, 820, 200
					make_text_macro 'E', area, 830, 200

					make_text_macro 'O', area, 850, 200
					make_text_macro 'V', area, 860, 200
					make_text_macro 'E', area, 870, 200
					make_text_macro 'R', area, 880, 200
				jmp aici99
				alb99:
				deseneaza_patratel_albastru 421,422
				inc nr
				aici99:
				add matrice[352],'0'
				make_text_macro matrice[352],area,435,430
		COLOANA_10_LINIA_9:
			cmp ecx,500
			jg afara
			cmp ecx,460
			jl afara
				cmp matrice[356],9
				jne alb910
				deseneaza_patratel_rosu 421,462
					make_text_macro 'G', area, 800, 200
					make_text_macro 'A', area, 810, 200
					make_text_macro 'M', area, 820, 200
					make_text_macro 'E', area, 830, 200

					make_text_macro 'O', area, 850, 200
					make_text_macro 'V', area, 860, 200
					make_text_macro 'E', area, 870, 200
					make_text_macro 'R', area, 880, 200
				jmp aici910
				alb910:
				deseneaza_patratel_albastru 421,462
				inc nr
				aici910:
				add matrice[356],'0'
				make_text_macro matrice[356],area,475,430
LINIA_10:
	cmp ebx,460
	jl afara
	cmp ebx,500
	jg afara
		COLOANA_1_LINIA_10:
			cmp ecx,100
			jl afara
			cmp ecx,140
			jg COLOANA_2_LINIA_10
				cmp matrice[360],9
				jne alb101
				deseneaza_patratel_rosu 461,102
					make_text_macro 'G', area, 800, 200
					make_text_macro 'A', area, 810, 200
					make_text_macro 'M', area, 820, 200
					make_text_macro 'E', area, 830, 200

					make_text_macro 'O', area, 850, 200
					make_text_macro 'V', area, 860, 200
					make_text_macro 'E', area, 870, 200
					make_text_macro 'R', area, 880, 200
				jmp aici101
				alb101:
				deseneaza_patratel_albastru 461,102
				inc nr
				aici101:
				add matrice[360],'0'
				make_text_macro matrice[360],area,115,470
				jmp afara
		COLOANA_2_LINIA_10:
			cmp ecx,180
			jg COLOANA_3_LINIA_10
			cmp ecx,140
			jl afara
				cmp matrice[364],9
				jne alb102
				deseneaza_patratel_rosu 461,142
					make_text_macro 'G', area, 800, 200
					make_text_macro 'A', area, 810, 200
					make_text_macro 'M', area, 820, 200
					make_text_macro 'E', area, 830, 200

					make_text_macro 'O', area, 850, 200
					make_text_macro 'V', area, 860, 200
					make_text_macro 'E', area, 870, 200
					make_text_macro 'R', area, 880, 200
				jmp aici102
				alb102:
				deseneaza_patratel_albastru 461,142
				inc nr
				aici102:
				add matrice[364],'0'
				make_text_macro matrice[364],area,155,470
		COLOANA_3_LINIA_10:
			cmp ecx,220
			jg COLOANA_4_LINIA_10
			cmp ecx,180
			jl afara
			cmp matrice[368],9
				jne alb103
				deseneaza_patratel_rosu 461,182
					make_text_macro 'G', area, 800, 200
					make_text_macro 'A', area, 810, 200
					make_text_macro 'M', area, 820, 200
					make_text_macro 'E', area, 830, 200

					make_text_macro 'O', area, 850, 200
					make_text_macro 'V', area, 860, 200
					make_text_macro 'E', area, 870, 200
					make_text_macro 'R', area, 880, 200
				jmp aici103
				alb103:
				deseneaza_patratel_albastru 461,182
				inc nr
				aici103:
				add matrice[368],'0'
				make_text_macro matrice[368],area,195,470
		COLOANA_4_LINIA_10:
			cmp ecx,260
			jg COLOANA_5_LINIA_10
			cmp ecx,220
			jl afara
				cmp matrice[372],9
				jne alb104
				deseneaza_patratel_rosu 461,222
					make_text_macro 'G', area, 800, 200
					make_text_macro 'A', area, 810, 200
					make_text_macro 'M', area, 820, 200
					make_text_macro 'E', area, 830, 200

					make_text_macro 'O', area, 850, 200
					make_text_macro 'V', area, 860, 200
					make_text_macro 'E', area, 870, 200
					make_text_macro 'R', area, 880, 200
				jmp aici104
				alb104:
				deseneaza_patratel_albastru 461,222
				inc nr
				aici104:
				add matrice[372],'0'
				make_text_macro matrice[372],area,235,470
		COLOANA_5_LINIA_10:
			cmp ecx,300
			jg COLOANA_6_LINIA_10
			cmp ecx,260
			jl afara
				cmp matrice[376],9
				jne alb105
				deseneaza_patratel_rosu 461,262
					make_text_macro 'G', area, 800, 200
					make_text_macro 'A', area, 810, 200
					make_text_macro 'M', area, 820, 200
					make_text_macro 'E', area, 830, 200

					make_text_macro 'O', area, 850, 200
					make_text_macro 'V', area, 860, 200
					make_text_macro 'E', area, 870, 200
					make_text_macro 'R', area, 880, 200
				jmp aici105
				alb105:
				deseneaza_patratel_albastru 461,262
				inc nr
				aici105:
				add matrice[376],'0'
				make_text_macro matrice[376],area,275,470
		COLOANA_6_LINIA_10:
			cmp ecx,340
			jg COLOANA_7_LINIA_10
			cmp ecx,300
			jl afara
				cmp matrice[380],9
				jne alb106
				deseneaza_patratel_rosu 461,302
					make_text_macro 'G', area, 800, 200
					make_text_macro 'A', area, 810, 200
					make_text_macro 'M', area, 820, 200
					make_text_macro 'E', area, 830, 200

					make_text_macro 'O', area, 850, 200
					make_text_macro 'V', area, 860, 200
					make_text_macro 'E', area, 870, 200
					make_text_macro 'R', area, 880, 200
				jmp aici106
				alb106:
				deseneaza_patratel_albastru 461,302
				inc nr
				aici106:
				add matrice[380],'0'
				make_text_macro matrice[380],area,315,470
		COLOANA_7_LINIA_10:
			cmp ecx,380
			jg COLOANA_8_LINIA_10
			cmp ecx,340
			jl afara
				cmp matrice[384],9
				jne alb107
				deseneaza_patratel_rosu 461,342
					make_text_macro 'G', area, 800, 200
					make_text_macro 'A', area, 810, 200
					make_text_macro 'M', area, 820, 200
					make_text_macro 'E', area, 830, 200

					make_text_macro 'O', area, 850, 200
					make_text_macro 'V', area, 860, 200
					make_text_macro 'E', area, 870, 200
					make_text_macro 'R', area, 880, 200
				jmp aici107
				alb107:
				deseneaza_patratel_albastru 461,342
				inc nr
				aici107:
				add matrice[384],'0'
				make_text_macro matrice[384],area,355,470
		COLOANA_8_LINIA_10:
			cmp ecx,420
			jg COLOANA_9_LINIA_10
			cmp ecx,380
			jl afara
				cmp matrice[388],9
				jne alb108
				deseneaza_patratel_rosu 461,382
					make_text_macro 'G', area, 800, 200
					make_text_macro 'A', area, 810, 200
					make_text_macro 'M', area, 820, 200
					make_text_macro 'E', area, 830, 200

					make_text_macro 'O', area, 850, 200
					make_text_macro 'V', area, 860, 200
					make_text_macro 'E', area, 870, 200
					make_text_macro 'R', area, 880, 200
				jmp aici108
				alb108:
				deseneaza_patratel_albastru 461,382
				inc nr
				aici108:
				add matrice[388],'0'
				make_text_macro matrice[388],area,395,470
		COLOANA_9_LINIA_10:
			cmp ecx,460
			jg COLOANA_10_LINIA_10
			cmp ecx,420
			jl afara
				cmp matrice[392],9
				jne alb109
				deseneaza_patratel_rosu 461,422
					make_text_macro 'G', area, 800, 200
					make_text_macro 'A', area, 810, 200
					make_text_macro 'M', area, 820, 200
					make_text_macro 'E', area, 830, 200

					make_text_macro 'O', area, 850, 200
					make_text_macro 'V', area, 860, 200
					make_text_macro 'E', area, 870, 200
					make_text_macro 'R', area, 880, 200
				jmp aici109
				alb109:
				deseneaza_patratel_albastru 461,422
				inc nr
				aici109:
				add matrice[392],'0'
				make_text_macro matrice[392],area,435,470
		COLOANA_10_LINIA_10:
			cmp ecx,500
			jg afara
			cmp ecx,460
			jl afara
				cmp matrice[396],9
				jne alb1010
				deseneaza_patratel_rosu 461,462
					make_text_macro 'G', area, 800, 200
					make_text_macro 'A', area, 810, 200
					make_text_macro 'M', area, 820, 200
					make_text_macro 'E', area, 830, 200

					make_text_macro 'O', area, 850, 200
					make_text_macro 'V', area, 860, 200
					make_text_macro 'E', area, 870, 200
					make_text_macro 'R', area, 880, 200
				jmp aici1010
				alb1010:
				deseneaza_patratel_albastru 461,462
				inc nr
				aici1010:
				add matrice[396],'0'
				make_text_macro matrice[396],area,475,470
	
	afara:
		inc nr
		cmp nr,90
		jne aicii
			make_text_macro 'S', area, 800, 200
			make_text_macro 'U', area, 810, 200
			make_text_macro 'C', area, 820, 200
			make_text_macro 'C', area, 830, 200
			make_text_macro 'C', area, 830, 200
			make_text_macro 'E', area, 840, 200
			make_text_macro 'S', area, 850, 200
			make_text_macro 'S', area, 860, 200
		
	aicii:
evt_timer:
	inc counter
	
afisare_litere:
	; ;afisam valoarea counter-ului curent (sute, zeci si unitati)
	; mov ebx, 10
	; mov eax, counter
	; ;cifra unitatilor
	; mov edx, 0
	; div ebx
	; add edx, '0'
	; make_text_macro edx, area, 30, 10
	; ;cifra zecilor
	; mov edx, 0
	; div ebx
	; add edx, '0'
	; make_text_macro edx, area, 20, 10
	; ;cifra sutelor
	; mov edx, 0
	; div ebx
	; add edx, '0'
	; make_text_macro edx, area, 10, 10
	
	;scriem un mesaj
	make_text_macro 'M', area, 50, 10
	make_text_macro 'I', area, 60, 10
	make_text_macro 'N', area, 70, 10
	make_text_macro 'E', area, 80, 10
	make_text_macro 'S', area, 90, 10
	make_text_macro 'W', area, 100, 10
	make_text_macro 'E', area, 110, 10
	make_text_macro 'E', area, 120, 10
	make_text_macro 'P', area, 130, 10
	make_text_macro 'E', area, 140, 10
	make_text_macro 'R', area, 150, 10


final_draw:
	popa
	mov esp, ebp
	pop ebp
	ret
draw endp

start:
	;alocam memorie pentru zona de desenat
	mov eax, area_width
	mov ebx, area_height
	mul ebx
	shl eax, 2
	push eax
	call malloc
	add esp, 4
	mov area, eax
	;apelam functia de desenare a ferestrei
	; typedef void (*DrawFunc)(int evt, int x, int y);
	; void __cdecl BeginDrawing(const char *title, int width, int height, unsigned int *area, DrawFunc draw);
	push offset draw
	push area
	push area_height
	push area_width
	push offset window_title
	call BeginDrawing
	add esp, 20
	
	;terminarea programului
	push 0
	call exit
end start
