``` abap
*&---------------------------------------------------------------------*
*& Include ZC102SDR0008TOP                          - Report ZC102SDR0008
*&---------------------------------------------------------------------*
REPORT zc102sdr0008 MESSAGE-ID zc102msg.

**********************************************************************
* Macro
**********************************************************************
DEFINE _init.

  REFRESH &1.
  CLEAR &1.

END-OF-DEFINITION.

**********************************************************************
*Tap Strip controls
**********************************************************************
CONTROLS gc_tab TYPE TABSTRIP.      " TAB Strip object

DATA : gv_subscreen TYPE sy-dynnr.  " Subscreen number

**********************************************************************
* Class Instance
**********************************************************************
DATA : go_tab_cont1 TYPE REF TO cl_gui_custom_container,  " 미검수 리스트
       go_tab_grid1 TYPE REF TO cl_gui_alv_grid.

DATA : go_tab_cont2 TYPE REF TO cl_gui_custom_container.

*-- TAB2에 붙을 컨테이너
DATA : go_split_cont TYPE REF TO cl_gui_splitter_container,
       go_left_cont  TYPE REF TO cl_gui_container,
       go_left_grid  TYPE REF TO cl_gui_alv_grid,
       go_right_cont TYPE REF TO cl_gui_container,
       go_right_grid TYPE REF TO cl_gui_alv_grid.

*-- For Popup 110
DATA : go_pop_cont TYPE REF TO cl_gui_custom_container,
       go_pop_grid TYPE REF TO cl_gui_alv_grid.

*-- For Popup 120
DATA : go_pop_cont2 TYPE REF TO cl_gui_custom_container,
       go_pop_grid2 TYPE REF TO cl_gui_alv_grid.

*-- Text editor용
DATA : go_text_edit TYPE REF TO cl_gui_textedit,
       go_text_cont TYPE REF TO cl_gui_custom_container.

**********************************************************************
* Internal table and Work area
**********************************************************************
*-- 긴급 판매오더 헤더 + 아이템
DATA : BEGIN OF gs_so,
         vbeln_so   TYPE zc102sdt0006-vbeln_so,   " H-판매오더 번호
         partner    TYPE zc102sdt0006-partner,    " H-비즈니스 파트너
         cusno      TYPE zc102sdt0006-cusno,      " H-고객 번호
         matnr      TYPE zc102sdt0007-matnr,      " I-자재번호
         maktx      TYPE zc102mmt0004-maktx,      " 자재명 FROM 자재마스터
         pequan     TYPE zc102sdt0007-pequan,     " I-주문수량
         meins      TYPE zc102sdt0007-meins,      " I-수량 단위
         netwr      TYPE zc102sdt0006-netwr,      " I-총 판매가
         waers      TYPE zc102sdt0006-waers,      " I-통화키
         audat      TYPE zc102sdt0006-audat,      " H-주문 일자
         delid      TYPE zc102sdt0006-delid,      " H-납품일
         stlno      TYPE zc102sdt0005-stlno,      " I-창고번호
         status     TYPE zc102sdt0007-status,     " I-긴급판매오더(생산 중 여부)
         state      TYPE icon-id,
         state_text TYPE char20,
         werks      TYPE zc102mmt0008-werks,
         regno      TYPE zc102mmt0008-regno,
         empnam     TYPE zc102hrt0002-empnam,  " 배송기사 GT_SO_NOT에서만 사용
         adrnr      TYPE zc102sdt0011-adrnr,   " 배송지 GT_SO_NOT에서만 사용
         check      TYPE c, " 납품 가능 여부(체크박스)
         cell_tab   TYPE lvc_t_styl, " 체크박스용
       END OF gs_so,
       gt_so     LIKE TABLE OF gs_so,
       gs_so_not LIKE gs_so,
       gt_so_not LIKE TABLE OF gs_so.

DATA : gt_soall TYPE TABLE OF zvsalesorder102,
       gs_soall TYPE zvsalesorder102.

*-- 납품오더 insert용
DATA : gs_do   TYPE zc102sdt0004,
       gs_do_i TYPE zc102sdt0005.

DATA : gt_werks TYPE TABLE OF zc102mmt0008,
       gs_werks TYPE zc102mmt0008.

*-- 긴급 납품오더 헤더(배송정보 미입력)
DATA : BEGIN OF gs_do_not,
         vbeln_del TYPE zc102sdt0004-vbeln_del, " 납품오더번호
         vbeln_so  TYPE zc102sdt0004-vbeln_so,  " 판매오더번호
         partner   TYPE zc102sdt0004-partner,   " 고객
         cusno     TYPE zc102sdt0004-cusno,     " 고객번호
         finalsp   TYPE zc102sdt0004-finalsp,   " 금액
         waers     TYPE zc102sdt0004-waers,     " 통화키
         vdatu     TYPE zc102sdt0004-vdatu,     " 납품 예정일
         leati     TYPE zc102sdt0011-leati,     " 배송 리드타임
         wadat_ist TYPE zc102sdt0004-wadat_ist, " 배송 시작 예정일
         vsbed     TYPE zc102sdt0011-vsbed,     " 배송 방법
         adrnr     TYPE zc102sdt0011-adrnr,     " 배송지
         gbstk     TYPE zc102sdt0004-gbstk,     " 배송 상태
         empno     TYPE zc102hrt0002-empno,     " 배송사원 번호
         del_char  TYPE zc102sdt0011-del_char,  " 배송 담당자
         telf1     TYPE zc102sdt0011-telf1,     " 연락처
         email     TYPE zc102hrt0002-email,
         stlno     TYPE zc102sdt0005-stlno,
         modi_yn,                               " 해당 레코드 변경 여부
         cell_tab  TYPE lvc_t_styl,             " ALV Edit style
       END OF gs_do_not,
       gt_do_not LIKE TABLE OF gs_do_not,
       gt_done   LIKE TABLE OF gs_do_not.

*-- 지역코드 도메인 값
DATA : gt_regno_d TYPE TABLE OF dd07v,
       gs_regno_d TYPE dd07v.

*-- 배송 정보 관리 테이블
DATA : gs_delivery TYPE zc102sdt0011.

*-- For ALV
DATA : gt_tab_fcat      TYPE lvc_t_fcat, " 긴급 납품 FCAT
       gs_tab_fcat      TYPE lvc_s_fcat,

       gt_left_fcat     TYPE lvc_t_fcat, " 검수 완료 리스트 FCAT
       gs_left_fcat     TYPE lvc_s_fcat,

       gt_right_fcat    TYPE lvc_t_fcat, " 긴급 배송 FCAT
       gs_right_fcat    TYPE lvc_s_fcat,

       gs_variant_tab   TYPE disvariant,
       gs_variant_left  TYPE disvariant,
       gs_variant_right TYPE disvariant,
       gs_variant_pop   TYPE disvariant,
       gs_variant_pop2  TYPE disvariant,

       gs_layout_tab    TYPE lvc_s_layo,
       gs_layout_left   TYPE lvc_s_layo,
       gs_layout_right  TYPE lvc_s_layo.

DATA : gt_ui_functions TYPE ui_functions,
       gs_button_tab1  TYPE stb_button,   " TAB1 납품오더 생성 버튼
       gs_button_tab2  TYPE stb_button.   " TAB2 배송 배정 버튼

*-- BP 정보 관리
DATA : gt_bp   TYPE TABLE OF zc102sdt0001,
       gs_bp   TYPE zc102sdt0001,
       gt_line TYPE TABLE OF zc102sdt0005,
       gs_line TYPE zc102sdt0005.

*-- 배송기사 정보
DATA : gt_driver_info TYPE TABLE OF zc102hrt0002, " 전체 기사 정보
       gs_driver_info TYPE zc102hrt0002,
       gt_driver      TYPE TABLE OF zc102hrt0002, " 인터널 테이블 반영용
       gs_driver      TYPE zc102hrt0002.

*-- 지역별 판매오더 수, 배송기사 수
DATA : BEGIN OF gs_regno,
         regno TYPE zc102mmt0008-regno,
         scnt  TYPE i,
         dcnt  TYPE i,
       END OF gs_regno,
       gt_regno LIKE TABLE OF gs_regno.

DATA : BEGIN OF gs_driver_check.
         INCLUDE STRUCTURE zc102hrt0002.

DATA :   check    TYPE c,          " 가용여부(체크박스)
         cell_tab TYPE lvc_t_styl, " 스타일
       END OF gs_driver_check,
       gt_driver_check LIKE TABLE OF gs_driver_check.

*-- 팝업 110 ALV용
DATA : gs_pop_fcat   TYPE lvc_s_fcat,
       gt_pop_fcat   TYPE lvc_t_fcat,
       gs_pop_layout TYPE lvc_s_layo.

*-- 팝업 120 ALV용
DATA : gs_pop_fcat2   TYPE lvc_s_fcat,
       gt_pop_fcat2   TYPE lvc_t_fcat,
       gs_pop_layout2 TYPE lvc_s_layo.

**********************************************************************
* Global variable
**********************************************************************
DATA : gv_okcode   TYPE sy-ucomm.

*-- 납품오더 생성
DATA : gv_delno TYPE zc102sdt0004-vbeln_del.

*-- 납품 처리 화면 스크린 조건
RANGES : gr_vbeln_so FOR zc102sdt0006-vbeln_so.
RANGES : gr_partner  FOR zc102sdt0006-partner.

DATA   : gv_bp TYPE zc102sdt0006-partner,
         gv_so TYPE zc102sdt0006-vbeln_so.

*-- 토글 버튼
DATA :gv_mode VALUE 'D'. "토글 전환용 변수

*-- 이메일 템플릿용
DATA gt_editor_lines TYPE TABLE OF tline.
