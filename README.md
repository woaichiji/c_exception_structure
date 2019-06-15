## 利用C 标准函数库 `setjmp.h` 和宏定义实现一个类似java一样的异常处理机制

#### 原理是：合理的运用了 `setjum` 和 `longjmp` 函数

### 基本的try-catch
```
 #include <stdio.h>
 #include <setjmp.h>

 #define TRY do{ jmp_buf ex_buf__; if( !setjmp(ex_buf__) ){
 #define CATCH } else {
 #define ETRY } }while(0)
 #define THROW longjmp(ex_buf__, 1)

 int
 main(int argc, char** argv)
 {
    TRY
    {
       printf("In Try Statement\n");
       THROW;
       printf("I do not appear\n");
    }
    CATCH
    {
       printf("Got Exception!\n");
    }
    ETRY;

    return 0;
 }

```

### 增加自定义异常类型

```
#include <stdio.h>
#include <setjmp.h>

#define TRY do{ jmp_buf ex_buf__; switch( setjmp(ex_buf__) ){ case 0:
#define CATCH(x) break; case x:
#define ETRY } }while(0)
#define THROW(x) longjmp(ex_buf__, x)

#define FOO_EXCEPTION (1)
#define BAR_EXCEPTION (2)
#define BAZ_EXCEPTION (3)

int
main(int argc, char** argv)
{
   TRY
   {
      printf("In Try Statement\n");
      THROW( BAR_EXCEPTION );
      printf("I do not appear\n");
   }
   CATCH( FOO_EXCEPTION )
   {
      printf("Got Foo!\n");
   }
   CATCH( BAR_EXCEPTION )
   {
      printf("Got Bar!\n");
   }
   CATCH( BAZ_EXCEPTION )
   {
      printf("Got Baz!\n");
   }
   ETRY;

   return 0;
}

```


### 增加 Finally 支持
```
#include <stdio.h>
#include <setjmp.h>

#define TRY do{ jmp_buf ex_buf__; switch( setjmp(ex_buf__) ){ case 0: while(1){
#define CATCH(x) break; case x:
#define FINALLY break; } default : {
#define ETRY break; } } }while(0)
#define THROW(x) longjmp(ex_buf__, x)

#define FOO_EXCEPTION (1)
#define BAR_EXCEPTION (2)
#define BAZ_EXCEPTION (3)

int
main(int argc, char** argv)
{
   TRY
   {
      printf("In Try Statement\n");
      THROW( BAR_EXCEPTION );
      printf("I do not appear\n");
   }
   CATCH( FOO_EXCEPTION )
   {
      printf("Got Foo!\n");
   }
   CATCH( BAR_EXCEPTION )
   {
      printf("Got Bar!\n");
   }
   CATCH( BAZ_EXCEPTION )
   {
      printf("Got Baz!\n");
   }
   FINALLY
   {
      printf("...et in arcadia Ego\n");
   }
   ETRY;

   return 0;
}
```
### 接下来就呈现一个完整的头定义

```
#ifndef _TRY_THROW_CATCH_H_
#define _TRY_THROW_CATCH_H_

#include <stdio.h>
#include <setjmp.h>

/* 来自
 * http://www.di.unipi.it/~nids/docs/longjump_try_trow_catch.html
 */

#define TRY do { jmp_buf ex_buf__; switch( setjmp(ex_buf__) ) { case 0: while(1) {
#define CATCH(x) break; case x:
#define FINALLY break; } default: {
#define ETRY break; } } }while(0)
#define THROW(x) longjmp(ex_buf__, x)

#endif /*!_TRY_THROW_CATCH_H_*/

```