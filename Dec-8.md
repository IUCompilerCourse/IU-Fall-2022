# Review for Final Exam

Overview of the topics on the final exam:

* Loops and Dataflow Analysis
* Tuples and Garbage Collection
* Functions

# Loops, Dataflow Analysis, and Mutable Variables

Main ideas:

* Cycles in the control-flow graph necessitate use of dataflow
  analysis during the liveness analysis of register allocation.

* Racket: Mutable variables require special care in Remove Complex Operands.

* Explicate control is responsible for lowering `while` loop into
  basic blocks and `goto`.


## Source Program

    (let ([x 0])
      (let ([i 0])
        (begin
          (while (eq? x (begin (set! x (read)) x))
            (set! i (+ i 1)))
          (+ i 40))))

## Uncover get!

    (let ([x57 0])
       (let ([i58 0])
          (begin 
             (while (eq? (get! x57) (begin (set! x57 (read))
                                           (get! x57)))
                (set! i58 (+ (get! i58) 1)))
             (+ (get! i58) 40))))

## Remove Complex Operands

    (let ([x57 0])
       (let ([i58 0])
          (begin 
             (while (let ([tmp59 x57])
                       (let ([tmp60 (begin (set! x57 (read))
                                           x57)])
                          (eq? tmp59 tmp60)))
                (set! i58
                   (let ([tmp61 i58])
                      (+ tmp61 1))))
             (let ([tmp62 i58])
                (+ tmp62 40)))))

## Explicate Control

    program:
    loop63:
        tmp59 = x57;
        x57 = (read);
        tmp60 = x57;
        if (eq? tmp59 tmp60)
           goto block65;
        else
           goto block64;
    block65:
        tmp61 = i58;
        i58 = (+ tmp61 1);
        goto loop63;
    block64:
        tmp62 = i58;
        return (+ tmp62 40);
    start:
        x57 = 0;
        i58 = 0;
        goto loop63;

# Uncover Live

    block65:
       (set 'x57 'i58)
    movq i58, tmp61
       (set 'tmp61 'x57)
    movq tmp61, i58
       (set 'x57 'i58)
    addq $1, i58
       (set 'x57 'i58)
    jmp loop63
       (set 'x57 'i58)

    start:
       (set)
    movq $0, x57
       (set 'x57)
    movq $0, i58
       (set 'x57 'i58)
    jmp loop63
       (set 'x57 'i58)

    block64:
       (set 'i58)
    movq i58, tmp62
       (set 'tmp62)
    movq tmp62, %rax
       (set)
    addq $40, %rax
       (set)
    jmp conclusion
       (set))

    loop63:
       (set 'x57 'i58)
    movq x57, tmp59
       (set 'i58 'tmp59)
    callq read_int
       (set 'i58 'tmp59)
    movq %rax, x57
       (set 'x57 'i58 'tmp59)
    movq x57, tmp60
       (set 'x57 'i58 'tmp60 'tmp59)
    cmpq tmp60, tmp59
       (set 'x57 'i58)
    je block65
       (set 'x57 'i58)
    jmp block64
       (set 'x57 'i58)


# Tuples and Garbage Collection


# Functions
