``` abap
*&---------------------------------------------------------------------*
*& Include ZC102SDR0002TOP                          - Report ZC102SDR0002
*&---------------------------------------------------------------------*
REPORT zc102sdr0002 MESSAGE-ID zc102msg.

**********************************************************************
* Macro
**********************************************************************
DEFINE _init.

  REFRESH &1.
  CLEAR &1.

END-OF-DEFINITION.

**********************************************************************
* TABLES
**********************************************************************
TABLES : zc102sdt0007, zc102sdt0006, zc102hrt0002.

**********************************************************************
* Class instance
**********************************************************************
DATA: go_main_cont  TYPE REF TO cl_gui_custom_container,    " 메인 컨테이너
      go_split_cont TYPE REF TO cl_gui_splitter_container,  " 메인 스플릿 컨테이너
      go_left_cont  TYPE REF TO cl_gui_container,           " 좌측 컨테이너
      go_right_cont TYPE REF TO cl_gui_container,           " 우측 컨테이너
      go_pop_cont1  TYPE REF TO cl_gui_custom_container.    " 팝업 컨테이너

DATA : go_left_grid  TYPE REF TO cl_gui_alv_grid,           " 좌측 ALV
       go_right_grid TYPE REF TO cl_gui_alv_grid,           " 우측 ALV
       go_pop_grid1  TYPE REF TO cl_gui_alv_grid.            " 팝업 ALV

**********************************************************************
* Internal table and Work area
**********************************************************************
*-- LEFT : 판매오더 Header
DATA : BEGIN OF gs_so_header,
         vbeln_so        TYPE zc102sdt0006-vbeln_so,   " 판매오더 번호
         cusno           TYPE zc102sdt0006-cusno,      " 고객 번호
         partner         TYPE zc102sdt0006-partner,    " 비즈니스 파트너
         ortype          TYPE zc102sdt0006-ortype,     " 판매오더 유형
         menge           TYPE zc102sdt0006-menge,      " 총 주문수량
         audat           TYPE zc102sdt0006-audat,      " 주문 일자
         delid           TYPE zc102sdt0006-delid,      " 납품일
         netwr           TYPE zc102sdt0006-netwr,      " 총 판매가
         waers           TYPE zc102sdt0006-waers,      " 통화키
         finalsp         TYPE zc102sdt0006-finalsp,    " 최종가격
         kbetr           TYPE zc102sdt0006-kbetr,      " 할인율
         dismo           TYPE zc102sdt0006-dismo,      " 할인가
         state           TYPE zc102sdt0006-state,      " 납품문서 생성 여부
         stax            TYPE zc102sdt0006-stax,       " 부가세
         isreg           TYPE zc102sdt0006-isreg.      " 정기/비정기
         INCLUDE STRUCTURE zc102cms0001.               " Timestamp
DATA :   status          TYPE icon-id,                 " 아이콘
         ortype_name(10),                              " 판매오더 유형 텍스트
         modi_yn,                                      " 수정 여부
         cell_tab        TYPE lvc_t_styl,              " 스타일 객체
       END OF gs_so_header,
       gt_so_header LIKE TABLE OF gs_so_header.

*-- For LEFT ALV
DATA : gs_fcat_left    TYPE lvc_s_fcat, " 필드 카탈로그 스트럭처
       gt_fcat_left    TYPE lvc_t_fcat, " 필드 카탈로그
       gs_layout_left  TYPE lvc_s_layo, " 레이아웃 설정
       gs_variant_left TYPE disvariant, " 레이아웃 설정
       gs_button       TYPE stb_button. " 툴바 버튼

*-- RIGHT : 판매오더 Item
DATA : BEGIN OF gs_so_item,
         matnr    TYPE zc102sdt0007-matnr,       " 자재 번호
         maktx    TYPE zc102mmt0004-maktx,       " 자재명
         pequan   TYPE zc102sdt0007-pequan,      " 수량
         meins    TYPE zc102sdt0007-meins,       " 단위
         scost    TYPE zc102sdt0007-scost,       " 판매가
         netwr    TYPE zc102sdt0007-netwr,       " 총 판매가
         waers    TYPE zc102sdt0007-waers,       " 통화키
         partner  TYPE zc102sdt0007-partner,     " 비즈니스 파트너
         cusno    TYPE zc102sdt0007-cusno,       " 고객 번호
         vbeln_so TYPE zc102sdt0007-vbeln_so.    " 판매오더 번호
         INCLUDE STRUCTURE zc102cms0001.         " Timestamp
DATA :   cell_tab TYPE lvc_t_styl,
         modi_yn,
       END OF gs_so_item,
       gt_so_item LIKE TABLE OF gs_so_item.

*-- For RIGHT ALV
DATA : gs_fcat_right    TYPE lvc_s_fcat, " 필드 카탈로그 스트럭처
       gt_fcat_right    TYPE lvc_t_fcat, " 필드 카탈로그
       gs_layout_right  TYPE lvc_s_layo, " 레이아웃 설정
       gs_variant_right TYPE disvariant. " 레이아웃 설정

*-- POPUP ALV
DATA : gs_popup LIKE LINE OF gt_so_item,
       gt_popup LIKE gt_so_item.

*-- For POPUP ALV
DATA : gs_fcat_pop1    TYPE lvc_s_fcat,   " 필드 카탈로그 스트럭처
       gt_fcat_pop1    TYPE lvc_t_fcat,   " 필드 카탈로그
       gs_layout_pop1  TYPE lvc_s_layo,   " 레이아웃 설정
       gs_variant_pop1 TYPE disvariant,   " 레이아웃 설정
       gt_ui_functions TYPE ui_functions. " 툴바 excluding

*-- For Search help
DATA : BEGIN OF gs_maktx,
         maktx TYPE zc102mmt0004-maktx, " 자재명
         matnr TYPE zc102mmt0004-matnr, " 자재 번호
       END OF gs_maktx,
       gt_maktx LIKE TABLE OF gs_maktx.

DATA : BEGIN OF gs_partner,
         partner TYPE zc102bpt0001-partner, " 고객번호
         name1   TYPE zc102sdt0001-name1,   " 고객명
       END OF gs_partner,
       gt_partner LIKE TABLE OF gs_partner.

DATA : BEGIN OF gs_empnam,
         empno  TYPE zc102hrt0002-empno,  " 사원 번호
         empnam TYPE zc102hrt0002-empnam, " 사원명
       END OF gs_empnam,
       gt_empnam LIKE TABLE OF gs_empnam.

**********************************************************************
* Global variable
**********************************************************************
DATA : gv_okcode TYPE sy-ucomm,
       gv_mode   VALUE 'D'.     " 편집 모드

*-- Screen header
DATA : gv_cnt1          TYPE i,                          " 납품생성 완료  건수
       gv_cnt2          TYPE i,                          " 납품생성 미완료 건수
       gv_cusno_checked TYPE abap_bool VALUE abap_false. " 고객 조회 성공 여부

*-- 고객 정보
DATA : gv_partner2 TYPE zc102bpt0001-partner,       " 검색 BP 번호
       gv_partner  TYPE zc102bpt0001-partner,       " BP 번호
       gv_cusno    TYPE zc102sdt0006-cusno,         " 고객 번호
       gv_name     TYPE zc102sdt0001-name1,         " 고객명
       gv_ctlzl    TYPE zc102sdt0001-ctlzl,         " 신용 평가
       gv_knkli    TYPE zc102sdt0001-knkli,         " 고객 등급
       gv_telf     TYPE zc102sdt0001-telf1,         " 연락처
       gv_stras    TYPE zc102sdt0001-stras,         " 주소

*-- 판매오더 가격 정보
       gv_total_sp TYPE zc102sdt0006-netwr,         " 총 판매가
       gv_dcrate   TYPE zc102sdt0006-kbetr,         " 할인율
       gv_dcprice  TYPE zc102sdt0006-dismo,         " 할인가
       gv_tax      TYPE zc102sdt0006-dismo,         " 부가세
       gv_finalsp  TYPE zc102sdt0006-finalsp,       " 최종 가격
       gv_cuky1    TYPE zc102sdt0006-waers,         " 통화키1
       gv_cuky2    TYPE zc102sdt0006-waers,         " 통화키2
       gv_cuky3    TYPE zc102sdt0006-waers,         " 통화키3
       gv_cuky4    TYPE zc102sdt0006-waers.         " 통화키4

*-- For POPUP
DATA : gv_sono  TYPE string,                      " 판매오더 번호
       gv_waers TYPE waers.                       " 고객 통화키

*-- For RIGHT
DATA : gv_vbeln_so TYPE zc102sdt0007-vbeln_so. " 판매오더 번호

**********************************************************************
* BATCH
**********************************************************************
DATA : BEGIN OF gs_batch.
         INCLUDE TYPE zc102mmt0019. " BP-자재 연결 테이블
DATA : END OF gs_batch,
gt_batch LIKE TABLE OF gs_batch.

DATA : BEGIN OF gs_cusmaster,
         cusno   TYPE zc102sdt0001-cusno,  " 고객 번호
         partner TYPE zc102sdt0001-partner," BP 번호
         knkli   TYPE zc102sdt0001-knkli,  " 고객 등급
         ctlzl   TYPE zc102sdt0001-ctlzl,  " 신용 평가
       END OF gs_cusmaster,
       gt_cusmaster LIKE TABLE OF gs_cusmaster.

DATA : BEGIN OF gs_kbetr,
         knkli TYPE zc102sdt0003-knkli, " 고객 등급
         ctlzl TYPE zc102sdt0003-ctlzl, " 신용 평가
         kbetr TYPE zc102sdt0003-kbetr, " 할인율
       END OF gs_kbetr,
       gt_kbetr LIKE TABLE OF gs_kbetr.

*-- For Batch
DATA : gv_date      TYPE sy-datum, " 날짜
       gv_valid_day,               " 배치 실행 날짜
       gv_field(10).
