# tiny-jit-assembler


```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/mman.h>
#include <stdlib.h>


int exec( unsigned char *code, int size ) {

  void *mem = mmap(NULL, size, PROT_WRITE | PROT_EXEC, MAP_ANON | MAP_PRIVATE, -1, 0);

  memcpy(mem, code, size);

  int (*func)() = mem;

  return func();
}

enum {
  EAX,
  EBX,
  ECX,
  EDX,
};

unsigned char asm_instruction[1000];
int asm_iter = 0;
void set_asm( unsigned char * ret, int size ) {
  for( int i = 0; i < size; i++ ) {
    asm_instruction[asm_iter++] = ret[i];
  } 
}

void mov( int reg, int val ) {
  unsigned char ret[5];

  switch( reg ) {
    case EAX :
      ret[0] = 0xB8;
      ret[1] = val;
      ret[2] = 0x00;
      ret[3] = 0x00;
      ret[4] = 0x00;
      break;

    case EBX : 
      ret[0] = 0xBB;
      ret[1] = val;
      ret[2] = 0x00;
      ret[3] = 0x00;
      ret[4] = 0x00;

      break;

    case EDX :
      ret[0] = 0xBA;
      ret[1] = val;
      ret[2] = 0x00;
      ret[3] = 0x00;
      ret[4] = 0x00;
      break;
  }


  set_asm( ret, sizeof( ret ) );
}

void push( int reg ) {
  unsigned char ret[1];
  switch( reg ) {
    case EAX :
      ret[0] = 0x50;
      break;
    case EBX :
      ret[0] = 0x53;
      break;
  }

  set_asm( ret, sizeof( ret ) );
}

void add( int reg, int reg1 ) {
  unsigned char ret[2];
  switch( reg ) {
    case EAX :
      ret[0] = 0x01;
      switch( reg1 ) {
        case EBX :
          ret[1] = 0xD8;
          break;
      }
      break; 
  }

  set_asm( ret, sizeof( ret ) );
}

void mul( int reg ) {
  unsigned char ret[2];
  switch( reg ) {
    case EAX :
      ret[0] = 0xF7;
      ret[1] = 0xE0;
      break;
    case EBX :
      ret[0] = 0xF7;  
      ret[1] = 0xE3;
      break;
  }

  set_asm( ret, sizeof( ret ) );
}

void pop( int reg ) {
  unsigned char ret[1];

  switch( reg ) {
    case EAX :
      ret[0] = 0x58;
      break;
    case EBX :
      ret[0] = 0x5B;
      break; 
  }

  set_asm( ret, sizeof( ret ) );

}

void ret() {
  unsigned char ret[1];
  ret[0] = 0xC3;
  set_asm( ret, sizeof( ret ) );
}

int main(int argc, char *argv[]) {

/*
4
+
2*3*5
+
6*7
+
4*1
*/

  mov( EAX, 2 );
  mov( EBX, 3 );
  mul( EBX );
  mov( EBX, 5 );
  mul( EBX );  
  push( EAX );


  mov( EAX, 6 );
  mov( EBX, 7 );
  mul( EBX );
  push( EAX );

  mov( EAX, 4 );
  mov( EBX, 1 );
  mul( EBX );
  push( EAX );

  pop( EAX );
  pop( EBX );
  add( EAX, EBX );
  pop( EBX );
  add( EAX, EBX );

  mov( EBX, 4 );
  add( EAX, EBX );
  ret();

  /*for( int i = 0; i < asm_iter; i++ ) {
    printf("%x ", asm_instruction[i] ); 
  }*/

  printf("%d\n", exec( asm_instruction, asm_iter ) );

  return 0;
}


```
