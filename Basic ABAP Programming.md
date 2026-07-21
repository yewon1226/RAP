## < 07/21 월 >
### < 단축키 정리 >
- Ctrl + Shift + F : pretty 정렬
- F9 : 출력
- Ctrl + F3 : 액티브

</br>
</br>
</br>

- 슈퍼클래스: 하나만 상속 가능(단일 상속), 인터페이스는 여러 개 구현 가능(다중 구현)
- 인터페이스: 기능을 연결하는 돼지코(규격/연결부) 역할

<img width="589" height="466" alt="image" src="https://github.com/user-attachments/assets/75e3f7c0-92f8-47f9-9f1b-6a2a0d90a85d" />
</br>
</br>
</br>
</br>

문제: 로컬 변수인데 `gv_` 를 사용함 → 로컬 변수는 `lv_` , 전역 변수는 `gv_` 를 사용해야 함
```abap
CLASS zcl_a06_w01 IMPLEMENTATION.
  METHOD if_oo_adt_classrun~main.
    DATA gv_name TYPE c LENGTH 10.      " lv_name
    gv_name = 'Kang'.      
    out->write( gv_name ).
  ENDMETHOD.
ENDCLASS.
```
</br>
</br>

- **`READ TABLE`** : 안전하게 조회하며, 대상이 없으면 `sy-subrc` 로 확인 가능(덤프 없음).
- **`itab[ ]`** : 코드가 간결하지만, 대상이 없으면 예외(덤프) 발생하며 `sy-subrc` 는 변경되지 않음.
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
    out->write( airline ).
  ENDMETHOD.
ENDCLASS.
```
</br>
</br>

- `CX` : Exception Class(예외 클래스)를 의미하며, 예외 처리에 사용
- `SE24` 에서 확인 가능
<img width="591" height="430" alt="image" src="https://github.com/user-attachments/assets/7775490f-dd06-405c-947a-778b98a635f9" />
</br>
</br>

- `TRY...CATCH` : 예외 발생 시 예외 객체를 자동 생성하여 처리
- 다형성 : 부모 타입으로 여러 자식 객체 사용 가능
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
</br>
</br>

- `NEW #( )` : CREATE OBJECT 대신 객체를 생성하는 최신 문법
```abap
" 기존
DATA(lo_obj) TYPE REF TO zcl_test.
CREATE OBJECT lo_obj.

" 최신
DATA(lo_obj) = NEW zcl_test( ).

" 타입 추론(#)
DATA lo_obj TYPE REF TO zcl_test.
lo_obj = NEW #( )
```
</br>
</br>

- 신문법
```abap
SELECT SINGLE FROM <테이블명>      " 단 건 조회
  FIELDS <필드1>, <필드2>, ...
  WHERE <조건>
  INTO @DATA(<구조체>).

SELECT FROM <테이블명>             " 다 건 조회
  FIELDS <필드1>, <필드2>, ...
  WHERE <조건>
  INTO TABLE @DATA(<내부테이블>).
```
```abap
CLASS zcl_a06_w02 IMPLEMENTATION.
  METHOD if_oo_adt_classrun~main.
    SELECT SINGLE FROM spfli
        FIELDS *
        WHERE carrid = 'AA'
        INTO @DATA(connections).
    out->write( connections ).
  ENDMETHOD.
ENDCLASS.
```
</br>
</br>

### Case 1
- VALUE는 구조체를 그대로 넣을 수 없으며, 필드명 = 값 형식으로 필드를 지정해야 
```abap
*   Case 1
    DATA ls_info LIKE connection.
    ls_info = value #( airpfrom = connection-airpfrom ).    " O (필드 지정)
*   DATA(ls_info) = value #( connection ).                  " X (구조체 전체 입력 → 오류)
```
</br>

### Case 2
- 인터널 테이블을 `VALUE` 로 생성하려면 테이블 타입에 키(`WITH EMPTY KEY` 또는 `WITH KEY`)를 정의해야 함
```abap
 TYPES: BEGIN OF ts_info,
             carrid TYPE spfli-carrid,
           END OF ts_info,
           tt_info TYPE TABLE OF ts_info WITH EMPTY KEY.

*    MOVE-CORRESPONDING connection TO data(ls_info).
*    DATA(ls_info) = CORRESPONDING ts_info( connection ).
    DATA(ls_info) = VALUE ts_info( carrid = connection-carrid ).             " O (구조체 생성)

    DATA lt_info TYPE tt_info.
    lt_info = VALUE #( ( carrid = 'AA' ) ( carrid = 'AB' ) ).                " O (타입 추론)
    DATA(lt_temp) = VALUE tt_info( ( carrid = 'AA' ) ( carrid = 'AB' ) ).    " O (타입 직접 지정)
    out->write( lt_temp ).
  ENDMETHOD.
```
</br>

### Case 3
- `ASSIGN` 으로 인터널 테이블 행을 필드 심볼에 연결하여 원본을 직접 변경
```abap
ASSIGN connections[ carrid = 'UA' connid = '3516' ]
    TO FIELD-SYMBOL(<fs_info>).
CLEAR <fs_info>.
out->write( connections ).
```
