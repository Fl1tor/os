# os

**Исследование компилятора GCC, языка ассемблера, процессов ОС, Makefile и Git**
# Лабораторная работа №1

## 1. Реализация функции на C++
### main.cpp с функцией
<img width="344" height="418" alt="изображение" src="https://github.com/user-attachments/assets/93f0f4f9-88fd-4acf-9f31-58fcc1c29685" />

### Компиляция и запуск
<img width="624" height="103" alt="изображение" src="https://github.com/user-attachments/assets/e3ac3613-1b0f-4bd6-b84b-c5853195f9e8" />

## 2. Компиляция в ассемблерный код
### Без оптимизации (-O0)
<img width="314" height="78" alt="изображение" src="https://github.com/user-attachments/assets/b376c2bb-91fd-4192-9825-56be533e59dd" />

```asm
.file	"main.cpp"  # имя исходного файла
	.text             # секция машинного кода
	.globl	_Z9factoriali  # объявление глобальной функции
	.def	_Z9factoriali;	.scl	2;	.type	32;	.endef
	.seh_proc	_Z9factoriali
_Z9factoriali:  # C++ имя: factorial(int)
.LFB2239:
	pushq	%rbp   # сохраняем старый указатель стека
	.seh_pushreg	%rbp
	movq	%rsp, %rbp   # устанавливаем новый указатель стека
	.seh_setframe	%rbp, 0
	subq	$16, %rsp  # выделяем 16 байт на стеке
	.seh_stackalloc	16
	.seh_endprologue
	movl	%ecx, 16(%rbp) # # n = ecx (первый целочисленный аргумент)
	movq	$1, -8(%rbp)  # result = 1
	movl	$1, -12(%rbp)    # i = 1
	jmp	.L2  # переход к проверке условия цикла
.L3:  # тело цикла
	movl	-12(%rbp), %eax # eax = i
	cltq  # Расширяем i до 64 бит для умножения
	movq	-8(%rbp), %rdx  # rdx = result
	imulq	%rdx, %rax  # rax = result * i
	movq	%rax, -8(%rbp) # result = result * i
	addl	$1, -12(%rbp) # i++
.L2: # проверка условия цикла
	movl	-12(%rbp), %eax   # eax = i
	cmpl	16(%rbp), %eax  # сравниваем i и n
	jle	.L3  # если i <= n, продолжаем
	movq	-8(%rbp), %rax # rax = result (возвращаемое значение)
	addq	$16, %rsp # кладем финальный res в rax (регистр возврата)
	popq	%rbp  # восстанавливаем стек
	ret  # возврат из функции
	.seh_endproc
	.section .rdata,"dr"
main:

	call	__main
	movl	$5, -4(%rbp) # n = 5
	movl	-4(%rbp), %eax # eax = n
	movl	%eax, %ecx  # ecx = n (аргумент для factorial)
	call	_Z9factoriali # factorial(n) # result = factorial(5)

```

### С оптимизацией (-O3)
<img width="321" height="43" alt="изображение" src="https://github.com/user-attachments/assets/d0c51cb1-4b37-4a82-a7a7-9cbb0f5903fa" />

```asm
.file	"main.cpp"
	.text
	.p2align 4
	.globl	_Z9factoriali
	.def	_Z9factoriali;	.scl	2;	.type	32;	.endef
	.seh_proc	_Z9factoriali
_Z9factoriali:
.LFB2263:
	.seh_endprologue
	testl	%ecx, %ecx # Проверяем n (аргумент лежит в %ecx)
	jle	.L4 # если n <= 0, вернуть 1

	leal	1(%rcx), %r8d  # r8d = n + 1 (граница)
	andl	$1, %ecx # ecx = n & 1 (четность)
	movl	$1, %eax # текущий множитель (начало)
	movl	$1, %edx # результат (начало)
	je	.L3 # Если n четное, сразу идем в цикл .L3
	movl	$2, %eax   # начинаем с 2 вместо 1
	cmpq	%r8, %rax  # проверка на выход
	je	.L1   # если 2 == n+1, выйти
	.p2align 5
	.p2align 4
	.p2align 3
.L3:
	imulq	%rax, %rdx # res *= i
	leaq	1(%rax), %rcx # Вычисляем (i + 1)
	addq	$2, %rax  # i += 2 (шаг цикла через два числа)
	imulq	%rcx, %rdx  # res *= (i + 1)
	cmpq	%r8, %rax  # достигли ли n?
	jne	.L3  # если нет, повторяем цикл
.L1:
	movq	%rdx, %rax  # результат в rax
	ret   # возврат
	.p2align 4,,10
	.p2align 3
 #  СЛУЧАЙ n <= 0
.L4:
	movl	$1, %edx # результат = 1
	movq	%rdx, %rax  # вернуть 1
	ret
	.seh_endproc
	.section .rdata,"dr"

main:
.LFB2264:
	pushq	%rbx
	.seh_pushreg	%rbx
	subq	$32, %rsp
	.seh_stackalloc	32
	.seh_endprologue
	call	__main
	movq	.refptr._ZSt4cout(%rip), %rcx # rcx = &cout
	movl	$5, %edx   # edx = 5
	call	_ZNSolsEi   # cout << 5
	movl	$2, %r8d
	leaq	.LC0(%rip), %rdx
	movq	%rax, %rbx
	movq	%rax, %rcx

	call	_ZSt16__ostream_insertIcSt11char_traitsIcEERSt13basic_ostreamIT_T0_ES6_PKS3_x
 #  ВЫВОД 120 ВЫЧИСЛЕНО КОМПИЛЯТОРОМ!
	movl	$120, %edx # edx = 120
	movq	%rbx, %rcx # rcx = cout
	call	_ZNSo9_M_insertIyEERSoT_  # cout << 120
	xorl	%eax, %eax
	addq	$32, %rsp
	popq	%rbx
	ret

.refptr._ZSt4cout:
	.quad	_ZSt4cout
```
## 3. Создание Makefile
###  Файл Makefile

<img width="327" height="358" alt="изображение" src="https://github.com/user-attachments/assets/cba1924a-87b6-4e65-bb91-1f0d92d1ebea" />

Команды:

  **make**
  
  **make run**
  
  **make asm**
  
  **make clean**

## 4. Усовершенствование программы

### Добавление параллельного процесса

Используем системный вызов fork().

<img width="585" height="382" alt="изображение" src="https://github.com/user-attachments/assets/2d7bf552-709a-43e7-9f3f-c226f5a85ba8" />

### Синхронизация процессов (pipe)

Для передачи данных между процессами используется канал pipe.

<img width="571" height="558" alt="изображение" src="https://github.com/user-attachments/assets/5de0df80-8d35-4076-859a-043ee950a9d2" />

