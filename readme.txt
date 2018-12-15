# Kofi Fort
# https://github.com/kofort9/lastproject--mips

.data 
  emptyMessage: .asciiz "Input is empty." # message that is displayed if string is empty 
  lengthMessage: .asciiz "Input is too long." # message that is displayed if string is too long
  baseErrorMessage: .asciiz "Invalid base-28 number." # message that is displayed if characters are not in set
  userInput: .space 1000
  
.text  

  main: # gets user input 
   li $v0, 8
   la $a0, userInput
   li $a1, 60
   syscall
 
  removeLeadingSpace: # removes unessary space in string
   li $t8, 32
   lb $t9, 0($a0)
   beq $t8, $t9, removeFirst
   move $t9, $a0
   j checkLength
   
  removeFirst: #removes first space in string
   addi $a0, $a0, 1
   j removeLeadingSpace
   
  checkLength: # checks input length
    addi $t0, $t0, 0 
    addi $t1, $t1, 10
    add $t4, $t4, $a0
    
  lengthLoop: # loops through each char in string
    lb $t2, 0 ($a0)
    beqz $t2, done
    beq $t2, $t1, done
    addi $a0, $a0, 1
    addi $t0, $t0, 1
    j lengthLoop
   
  done:
    beqz $t0, emptyError
    slti $t3, $t0, 5
    beqz $t3, lengthError
    move $a0, $t4
    j checkString
  
  emptyError: # message
    li $v0, 4
    la $a0, emptyMessage
    syscall 
    j exit 
    
  lengthError: # message
    li $v0, 4
    la $a0, lengthMessage
    syscall 
    j exit 
    
  checkString: # checks if input is valid
   lb $t5, 0($a0)
   beqz $t5, convert
   beq  $t5, $t1, convert
   slti $t6, $t5, 48 # if char < ascii(48),  input invalid,   ascii(48) = 0
   bne $t6, $zero, baseError
   slti $t6, $t5, 58 # if char < ascii(58),  input is valid,  ascii(58) = 9
   bne $t6, $zero, Increment
   slti $t6, $t5, 65 # if char < ascii(65),  input invalid,   ascii(65) = A
   bne $t6, $zero, baseError
   slti $t6, $t5, 82 # if char < ascii(88),  input is valid,  ascii(82) = R
   bne $t6, $zero, Increment
   slti $t6, $t5, 97 # if char < ascii(97),  input invalid,   ascii(97) = a
   bne $t6, $zero, baseError
   slti $t6, $t5, 115 # if char < ascii(115), input is valid, ascii(115) = r
   bne $t6, $zero, Increment
   bgt $t5, 114, baseError # if char > ascii(114), input invalid,  ascii(114) = s
   
  Increment: # increment checked letter to next 
   addi $a0, $a0, 1
   j checkString
   
  baseError: # message
   li $v0, 4
   la $a0, baseErrorMessage
   syscall 
   j exit
   
  convert: # convert string to decimals 
   move $a0, $t4
   addi $t7, $t7, 0
   add $s0, $s0, $t0
   addi $s0, $s0, -1
   li $s3, 3
   li $s2, 2
   li $s1, 1
   li $s5, 0
   
  convertString: #displays result and converts upper and lowers to corect numbers
   lb $s4, 0($a0)
   beqz $s4, displayResult
   beq $s4, $t1, displayResult
   slti $t6, $s4, 58
   bne $t6, $zero, Nums
   slti $t6, $s4, 82
   bne $t6, $zero, Upletters
   slti $t6, $s4, 115
   bne $t6, $zero, Lowletters
   
  Nums:
   addi $s4, $s4, -48
   j nextStep
   
  Upletters:
   addi $s4, $s4, -55
   j nextStep
   
  Lowletters:
   addi $s4, $s4, -87
   
  nextStep: # executes power functions
   beq $s0, $s3, threePow
   beq $s0, $s2, twoPow
   beq $s0, $s1, onePow
   beq $s0, $s5, zeroPow
   
  threePow:
   li $s6, 21952
   mult $s4, $s6
   mflo $s7
   add $t7, $t7, $s7
   addi $s0, $s0, -1
   addi $s0, $a0, 1
   j convertString
   
  twoPow:
   li $s6, 784
   mult $s4, $s6
   mflo $s7
   add $t7, $t7, $t7
   addi $s0, $s0, -1
   addi $a0, $a0, 1
   j convertString
   
  onePow:
   li $s6, 28
   mult $s4, $s6
   mflo $s7
   add $t7, $t7, $t7
   addi $s0, $s0, -1
   addi $a0, $a0, 1
   j convertString
   
  zeroPow:
   li $s6, 1
   mult $s4, $s6
   mflo $s7
   add $t7, $t7, $s7
   
  displayResult: # displays results
   li $v0, 1
   move $a0, $t7
   syscall  
    
  exit:
    li $v0, 10
    syscall 
 
 