# Assembly


; Calculate the Fibonacci number using recursion
; fib(n) = fib(n-1) + fib(n-2)


**lib.asm**
```Assembly

%include "constants.asm"

section .data

section .bss
	;buffer	resb	255			; reserverd 255 bytes

section .text

	; -----------------------------------------------------------------------------
	; Calculate the length of a string in rsi.
	; The string can end in LF (\n) or NUL (\0)
	; Input:
	;	rsi points to string
	; Output:
	;	rax contains length.
	; Error:
	;	rax returns -1
	_fn_strlen:
		xor		rax, rax			; clear rax

		_fn_strlen_loop:

			cmp		BYTE [rsi + rax], 0x0A	; compare with LF '\n'
			je		_fn_strlen_break
			cmp		BYTE [rsi + rax], 0x00	; compare with NUL '\0'
			je		_fn_strlen_break

			inc		rax

			jmp		_fn_strlen_loop

		_fn_strlen_break:
			ret

		_fn_strlen_error:
			xor		rax, rax			; set to 0 (all 0s)
			not		rax					; invert (all 1s)
			ret

	; -----------------------------------------------------------------------------
	;
	; Convert string of numbers to integer
	; Input:
	;	rsi points to the string of numbers ending in '\n'
	; Output:
	;	rcx contains the integer representation
	; Error:
	;	rax equals 1	
	; Modifies:
	;	rax, rbx, r8
	; Dependencies:
	;	_fn_strlen, to calculate the length  of the string
	;	_fn_exit, to exit if error
	_fn_str2int:
		xor		rbx, rbx			; clear rbx, to store the decimal part
		xor		rcx, rcx			; clear rcx, to store the final number
		mov		r8,	0x0A			; base 10

		_fn_str2int_init:
			; Input rsi points to string
			; Output rax contains length
			; Modifies rax
			call	_fn_strlen		; calculate the length of the string

			cmp		al, 0			; exit function if length = 0
			je		_fn_str2int_end		

			cmp		al, -1
			je		_fn_str2int_err

			mov		rdx, rax		; copy the length from rax to rdx
			mov		rax, 0x01		; put 1 in rax

		_fn_str2int_base:				; create base 10 position
			dec		dl
			cmp		dl, 0
			jle		_fn_str2int_decimal
			imul	rax, r8
			jmp		_fn_str2int_base

		_fn_str2int_decimal:
			mov		bl, byte [rsi]

			cmp		bl, 0x30		; compare with num. 0
			jl		_fn_str2int_err	; if not an integer go to erro
			cmp		bl, 0x39		; compare with num. 9
			jg		_fn_str2int_err	; if not an integer go to error

			sub		bl, 0x30		; to get integer representation
									; from a char number in ASCII
			mul		rbx				; add the integer part
			add		rcx, rax		; store decimal value

			inc		rsi

			jmp		_fn_str2int_init

		_fn_str2int_end:
			ret

		_fn_str2int_err:
			mov		rax, 0x01			; error code = 1
			ret

	; -----------------------------------------------------------------------------
	;
	; Convert an integer to string
	; Input:
	;	rax contains the integer to convert to string
	; Output:
	;	rsi points to the string
	;	rdx	contains the length of the string
	; Error:
	;	rax equals 1
	; Modifies:
	;	rax, rcx
	; Dependencies:
	;	_fn_strlen, to calculate the length  of the string
	;	_fn_exit, to exit if error
	; Notes:
	;	- Division:
	;		- Byte/Byte: The nominator resides in the AL register and AH is set to zero. After division,
	;					the instruction stores quotient in AL and the remainder in AH register. 
	;		- Word/Word: The AX register holds the numerator. After division, the quotient is stored in
	;					the AX register and the remainder goes to the DX register.
	;		- Word/Byte: The numerator is a 16-bit word stored in AX which is divided with an 8-bit
	;					denominator. After division, the AL contains the quotient and AH will contain
	;					the remainder.
	;		- Double Word/Word:  AX and DX stores the numerator. The most significant part resides in
	;					the DX register and the least significant bits of numerator are in the AX register.
	;					After the execution of DIV instruction, the remainder goes to DX register
	;					and the quotient lie in AX register.
	_fn_int2str:
		mov		rbx, 0x0A				; Put 10 in rbx
		xor		rcx, rcx				; clear rcx, to store the legnth
		xor		rdx, rdx				; clear rdx, to store the reminder for word or double word division

		_fn_int2str_init:
			cmp		rax, rbx				; check if number is smaller than the base
			jl		_fn_int2str_end			; end if smaller than base
			div		rbx						; rax = rax / rbx, rdx = rax % rbx
			add		dl, 0x30				; convert reminder to string in ASCII
			push	rdx						; push rdx to the stack
			inc		cl						; increment length
			jmp		_fn_int2str_init		; start again

		_fn_int2str_end:
			add		al, 0x30				; convert num to string
			push	rax
			inc		cl						; increment string length

			mov		al,	0x0A				; store LF '\n'
			push	rax
			inc		cl						; increment string length

		_fn_int2str_reverse:
			mov		dl, cl					; move the length to rdx
			xor		rcx, rcx
			cmp		dl, cl
			je		_fn_int2str_exit
			pop		rax
			mov		byte[buffer + rcx], al
			inc		cl
			jmp		_fn_int2str_reverse
			ret

		_fn_int2str_exit:
			mov		rsi, buffer				; point rsi to buffer address
			ret

		_fn_int2str_err:
			mov		rax, 0x01				; error code = 1
			ret

	; -----------------------------------------------------------------------------
	; Exit program
	; Input
	;	rdi contains the return code
	_fn_exit:
		; exit 
		mov		rax, SYS_EXIT		; syscall for exit
		syscall						; call kernel

	; -----------------------------------------------------------------------------
	; Reads input from stdin
	; reads from stdin
	; Input rsi -> buffer
	;		rdx = buffer length
	; modifies rax, rdi
	_fn_read:
		mov		rax, SYS_READ		; syscall for read
		mov		rdi, STDIN			; stdin file descriptor
		syscall						; execute read(0, buffer, bufsize)
		ret

	; -----------------------------------------------------------------------------
	; Writes to stdout
	; Input rsi -> string
	;       rdx = string length
	; Modifies rax, rdi
	_fn_write:
		mov		rax, SYS_WRITE		; syscall for write
		mov		rdi, STDOUT			; stdout file descriptor
		syscall						; execute write()
		ret
```

**constants.asm**
```Assembly
section .data

	SYS_EXIT	equ	60		; syscall number for exit (0x3c)
	SYS_WRITE	equ 1		; syscall number for write
	SYS_READ	equ 0		; syscall number for read
	STDIN		equ 0		; stdin file descriptor
	STDOUT		equ 1		; stdout file descriptor

section .bss

section .text
```

**prog.asm***
```Assembly
%include "lib.asm"

section .data

	msg1	db		"Enter  a number: ", 0
	msg1len	equ		$-msg1-1		; string length, minus NULL
	msg2	db		"Fib(n) = ", 0
	msg2len	equ		$-msg2-1		; string length, minus NULL

	bufsize	equ	10			; length of input buffer
	
	FIB_0	db	0			; fib(0) = 0
	FIB_1	db	1			; fib(1) = 1
							; ...
							; fib(2) = fib(1) + fib(0) = 1
							; fib(3) = fib(2) + fib(1) = fib(1) + fib(0) + 1 = 1 + 0 + 1 = 2
							; fib(4) = fib(3) + fib(2) = 2 + 1 = 3 
							; fib(n) = fib(n-1) + fib(n-2)

	fmt		db	"Fib(n)=%ld", 10, 0

section .bss
	buffer	resb	bufsize+1		; provide space for ending 0
	fib		resb	1

section .text
	global _start

	_start:

		; write to stdout
		mov		rsi, msg1			; address of string
		mov		rdx, msg1len		; length of string
		call	_fn_write

		; read stdin
		mov		rsi, buffer			; address of buffer
		mov		rdx, bufsize		; length of buffer
		call	_fn_read

		; convert string to integer
		call	_fn_str2int

		; calculates fibonacci number
		xor		rax, rax			; clear return value
		mov		rdx, rcx			; move 'n' to dl
		call	_fn_fib

		; convert result to string
		call	_fn_int2str

		; write to stdout
		call	_fn_write

		; exit program
		xor		rdi, rdi			; error code = 0
		call	_fn_exit


	; Calculate the Fibonacci number 'n'
	; Input:
	;	rdx
	; Output:
	;	rax
	; Modifies:
	;	rdx
	; push n-1 to stack
	; push n-2 to stack
	_fn_fib:
		push	rbp
		mov		rbp, rsp

		cmp		dl, 0				; input == 0 ?
		jne		_fn_fib_A
		add		al, byte [FIB_0]

		leave
		ret

	_fn_fib_A:
		cmp		dl, 1				; input == 1 ?
		jne		_fn_fib_B
		add		al, byte [FIB_1]

		leave
		ret

	_fn_fib_B:
		dec		dl					; calculate n-1
		push	rdx					; store n-1 on the stack

		call	_fn_fib				; recursion n-1

		pop		rdx					; load n-1 into rdx
		dec		dl					; calculate n-2

		cmp		dl, 0
		call	_fn_fib				; recursion n-2

		leave
		ret
```

Makefile
```Makefile
# $@ = target file
# $< = first dependency
# $^ = all dependencies

APP=prog

# First rule is the one executed when no parameters are fed to the Makefile
#all: run

#run: $(APP)
#	./$<

#run: $(APP)
#	qemu-system-x86_64 -fda $<
#
$(APP): $(APP).o
	ld -o $@ $<
#	gcc -Wall -no-pie -z noexecstack -o $@ $<

$(APP).o: $(APP).asm
	nasm -f elf64 -F dwarf $<

.PHONY: clean

clean:
	$(RM) $(APP) *.o
```

