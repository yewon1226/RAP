## < 07/21 월 >
### < 단축키 정리 >
- Ctrl + Shift + F : pretty 정렬
- F9 : 출력
- Ctrl + F3 : 액티브

</br>
</br>

- 슈퍼클래스: 하나만 상속 가능(단일 상속), 인터페이스는 여러 개 구현 가능(다중 구현)
- 인터페이스: 기능을 연결하는 돼지코(규격/연결부) 역할

<img width="589" height="466" alt="image" src="https://github.com/user-attachments/assets/75e3f7c0-92f8-47f9-9f1b-6a2a0d90a85d" />

</br>
</br>

문제: 로컬 변수인데 gv_를 사용함 → 로컬 변수는 `lv_` , 전역 변수는 `gv_` 를 사용해야 함
```abap
CLASS zcl_a06_w01 IMPLEMENTATION.
  METHOD if_oo_adt_classrun~main.
    DATA gv_name TYPE c LENGTH 10.      " lv_name
    gv_name = 'Kang'.      
    out->write( gv_name ).
  ENDMETHOD.
ENDCLASS.
```

- `READ TABLE` : 안전(덤프 없음), 시스템 변수(sy-subrc, sy-tabix) 변경됨.
- `airlines[ 10 ]` : 간결함, 없는 인덱스 접근 시 덤프 발생, 시스템 변수 변경 안 됨.
```abap
CLASS zcl_a06_w01 IMPLEMENTATION.
  METHOD if_oo_adt_classrun~main.
    SELECT carrid, carrname, currcode, 'Kang' AS testField FROM scarr
        INTO TABLE @DATA(airlines)
        WHERE currcode = 'USD'.

*    READ TABLE airlines INDEX 2 INTO DATA(airline).
    DATA(airline) = airlines[ 10 ].
    IF sy-subrc <> 0.
      out->write( 'Test' ).
    ENDIF.
*    out->write( sy-subrc ).
*    out->write( sy-tabix ).
    out->write( airline ).
  ENDMETHOD.
ENDCLASS.
```

- cx : exception class
- `SE24` 에서 확인 가능
<img width="591" height="430" alt="image" src="https://github.com/user-attachments/assets/7775490f-dd06-405c-947a-778b98a635f9" />


- 다형성 : 하나의 타입(부모)이 여러 가지 형태(자식)로 변하거나 동작할 수 있는 성질
```abap
TRY.
    < 예외가 발생할 수 있는 실행 코드 >
  CATCH <예외_클래스명> INTO DATA(<변수명>).
    < 예외 발생 시 처리할 코드 >
ENDTRY.
```
```abap
CLASS zcl_a06_w01 IMPLEMENTATION.
  METHOD if_oo_adt_classrun~main.
    SELECT carrid, carrname, currcode, 'Kang' AS testField FROM scarr
        INTO TABLE @DATA(airlines)
        WHERE currcode = 'USD'.

    TRY.
        DATA(airline) = airlines[ 10 ].
      " [자식 예외] 테이블 인덱스 에러 전용 클래스
*      CATCH cx_sy_itab_line_not_found INTO DATA(lo_ex).
      " [부모 예외] cx_sy_itab_line_not_found의 부모 클래스 (상속 관계)
      " 상속받은 부모 타입으로도 자식 에러를 잡아낼 수 있어 위 주석과 결과가 동일함
      CATCH cx_dynamic_check INTO DATA(lo_ex).
        DATA(error_mesg) = lo_ex->get_text(  ).
        out->write( error_mesg ).
        RETURN.
    ENDTRY.
    out->write( airline ).
  ENDMETHOD.
ENDCLASS.
```
