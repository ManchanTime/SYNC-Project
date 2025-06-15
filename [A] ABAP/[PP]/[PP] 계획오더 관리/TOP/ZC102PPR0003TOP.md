``` abap
*&---------------------------------------------------------------------*
*& Include ZC102PPR0003TOP                          - Report ZC102PPR0003
*&---------------------------------------------------------------------*
REPORT zc102ppr0003 MESSAGE-ID zc102msg.

**********************************************************************
* TABLES
**********************************************************************
TABLES :zc102ppt0002, zc102ppt0006, zc102ppt0012, zc102hrt0002.

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
* CLASS INSTANCE
**********************************************************************
DATA : go_container   TYPE REF TO cl_gui_custom_container,
       go_alv_grid    TYPE REF TO cl_gui_alv_grid,
       go_text_editer TYPE REF TO cl_gui_textedit,

       go_tab_cont1   TYPE REF TO cl_gui_custom_container,
       go_tab_grid1   TYPE REF TO cl_gui_alv_grid,
       go_tab_cont2   TYPE REF TO cl_gui_custom_container,
       go_tab_grid2   TYPE REF TO cl_gui_alv_grid,
       go_tab_cont3   TYPE REF TO cl_gui_custom_container,
       go_tab_grid3   TYPE REF TO cl_gui_alv_grid.


DATA : go_pop_cont TYPE REF TO cl_gui_custom_container, " 반려 POP-UP
       go_pop_grid TYPE REF TO cl_gui_alv_grid.


*-- Text editor용
DATA : go_text_edit TYPE REF TO cl_gui_textedit,
       go_text_cont TYPE REF TO cl_gui_custom_container.



**********************************************************************
*internal table and workarea
**********************************************************************

DATA : BEGIN OF gs_data,                "상단
         ostatus     TYPE icon-id.
         INCLUDE STRUCTURE zc102ppt0006.
DATA :   reason(30)  TYPE c,
*         reason_text(30) TYPE c,
         resonbutton TYPE  icon-id,        "반려 사유 텍스트 에디터 버튼용
         cell_tab    TYPE lvc_t_styl,
         modi_yn,
         sort_key,
       END OF gs_data,
       gt_data   LIKE TABLE OF gs_data,
       gt_save   LIKE TABLE OF gs_data,
       gt_backup LIKE TABLE OF gs_data,
       gs_backup LIKE gs_data.

DATA : BEGIN OF gs_tab1,                "전체 조회 TAB
         ostatus     TYPE icon-id.
         INCLUDE STRUCTURE zc102ppt0006.
DATA :   reason(30)  TYPE c,
*         reason_text(30) TYPE c,
         resonbutton TYPE  icon-id,            "반려 사유 텍스트 에디터 버튼용
         cell_tab    TYPE lvc_t_styl,
         modi_yn,
         sort_key,
       END OF gs_tab1,
       gt_tab1 LIKE TABLE OF gs_tab1.



DATA : BEGIN OF gs_tab2,                "승인 조회 TAB
         ostatus     TYPE icon-id.
         INCLUDE STRUCTURE zc102ppt0006.
DATA :   reason(30)  TYPE c,
*         reason_text(30) TYPE c,
         resonbutton TYPE icon-id,      "반려 사유 텍스트 에디터 버튼용
         cell_tab    TYPE lvc_t_styl,
         modi_yn,
         sort_key,
       END OF gs_tab2,
       gt_tab2 LIKE TABLE OF gs_tab2.


DATA : BEGIN OF gs_tab3,                "반려 조회 TAB
         ostatus     TYPE icon-id.
         INCLUDE STRUCTURE zc102ppt0006.
DATA :   reason(30)  TYPE c,
*         reason_text(30) TYPE c,
         resonbutton TYPE  icon-id,             "반려 사유 텍스트 에디터 버튼용
         cell_tab    TYPE lvc_t_styl,
         modi_yn,
         sort_key,
       END OF gs_tab3,
       gt_tab3 LIKE TABLE OF gs_tab3.

DATA : gt_delt TYPE TABLE OF zc102ppt0006, " 삭제용 Internal table
       gs_delt TYPE zc102ppt0006.


DATA : gt_approved TYPE TABLE OF zc102ppt0006, " 승인된 데이터 모아서 저장시 모두 생산오더 생성
       gs_approved TYPE zc102ppt0006.


DATA : gt_reject TYPE TABLE OF zc102ppt0006, " 반려된 데이터 모아서 저장시 모두 생산오더 생성
       gs_reject TYPE zc102ppt0006.

DATA: gt_route TYPE TABLE OF zc102ppt0010, "라우팅 테이블
      gs_route TYPE zc102ppt0010.


DATA: gt_mroute TYPE TABLE OF zc102ppt0009, "라우팅 마스터테이블
      gs_mroute TYPE zc102ppt0009.

DATA : BEGIN OF gs_pop,                "반려 사유 POP
         ostatus     TYPE icon-id.
         INCLUDE STRUCTURE zc102ppt0006.
DATA :   resonbutton TYPE  icon-id,        "반려 사유 텍스트 에디터 버튼용
*      reason(30)  TYPE c,
*         reason_text(50) TYPE c,       "반려상세내역
         cell_tab    TYPE lvc_t_styl,
         modi_yn,
       END OF gs_pop,
       gt_pop LIKE TABLE OF gs_pop.

*--plant 번호
DATA : BEGIN OF gs_plant,
         stlno TYPE zc102ppt0002-stlno,
         werks TYPE zc102ppt0002-werks,       " plant 마스터 테이블 / plant 번호
         wkcno TYPE zc102ppt0002-wkcno,       "workcenter 번호
       END OF gs_plant,
       gt_plant LIKE TABLE OF gs_plant.



*-- For ALV List Box
DATA : gt_list TYPE TABLE OF zc102ppt0006,
       gs_list TYPE zc102ppt0006.


*--Search Help 계획오더번호
DATA : BEGIN OF gs_plono,
         plono TYPE zc102ppt0006-plono,
       END OF gs_plono,
       gt_plono LIKE TABLE OF gs_plono.


*-- For ALV
DATA : gt_fcat          TYPE lvc_t_fcat, " TOP Field catalog
       gs_fcat          TYPE lvc_s_fcat,
       gs_top_layout    TYPE lvc_s_layo,
       gs_bottom_layout TYPE lvc_s_layo,
       gs_pop_layout    TYPE lvc_s_layo,
       gs_style         TYPE lvc_s_styl,

       gt_tab_fcat1     TYPE lvc_t_fcat,   " 탭 1 FCAT
       gt_tab_fcat2     TYPE lvc_t_fcat,   " 탭 2 FCAT
       gt_tab_fcat3     TYPE lvc_t_fcat,   " 탭 3 FCAT
       gs_tab_fcat1     TYPE lvc_s_fcat,
       gs_tab_fcat2     TYPE lvc_s_fcat,
       gs_tab_fcat3     TYPE lvc_s_fcat,
       gs_variant       TYPE disvariant.

DATA : gt_pop_fcat TYPE lvc_t_fcat, " 검수 POP-UP FCAT
       gs_pop_fcat TYPE lvc_s_fcat.


DATA : gs_sort TYPE lvc_s_sort,     " 정렬
       gt_sort TYPE lvc_t_sort.

DATA : gv_save_flag.                 " 승인 버튼 누르고 저장버튼 눌렀을 때 메세지


*-- For ALV List box
DATA : gt_drop TYPE lvc_t_drop,       "반품사유 list box
       gs_drop TYPE lvc_s_drop.

*--사원 이름 SEARCH HELP
DATA : BEGIN OF gs_empno,
         empno  TYPE zc102hrt0002-empno,
         empnam TYPE zc102hrt0002-empnam,
       END OF gs_empno,
       gt_empno LIKE TABLE OF gs_empno.

DATA : gt_memo      TYPE STANDARD TABLE OF string WITH EMPTY KEY.  " 메모 내용 저장

DATA : gt_roid  TYPE lvc_t_roid,
       gs_roid  TYPE lvc_s_roid,
       gt_roid2 TYPE lvc_t_roid,
       gs_roid2 TYPE lvc_s_roid.

DATA : gt_ui_functions TYPE ui_functions,
       gs_button       TYPE stb_button.

DATA: gt_sort_clear TYPE lvc_t_sort,       "아래 탭에 새로 생긴거 맨 위로 정렬
      gs_sort_clear TYPE lvc_s_sort.

**********************************************************************
* SCREEN ELEMENT
**********************************************************************
DATA : gv_psttr_fr TYPE zc102ppt0006-psttr,
       gv_psttr_to TYPE zc102ppt0006-psttr,
       gv_cnt1     TYPE i,  "전체
       gv_cnt2     TYPE i,
       gv_cnt3     TYPE i,
       gv_cnt4     TYPE i.


RANGES gr_psttr FOR zc102ppt0006-psttr.

**********************************************************************
* COMMON VARIABLE
**********************************************************************
DATA : gv_okcode TYPE sy-ucomm,
       gv_mode   VALUE 'D',
       gv_lock   TYPE bool,
       gv_answer,
       gv_empnam TYPE zc102hrt0002-empnam,
       gv_empno TYPE zc102hrt0002-empno,
       gv_plono  TYPE zc102ppt0006-plono.   " Search help

*--For pop up
DATA : gv_name TYPE sy-uname,
       gv_date TYPE c LENGTH 10,     " 문자열 날짜 포맷 (예: 2025.04.22)
       gv_time TYPE c LENGTH 8.      " 문자열 시간 포맷 (예: 10:32:45)

*--생산오더 번호 채번
DATA : gv_pdono TYPE zc102ppt0012,
       gs_pdono TYPE zc102ppt0012,
       gt_pdono TYPE TABLE OF zc102ppt0012.


DATA: BEGIN OF gs_text, "반려 사유를 입력하기 위한 전역 변수 선언 ( 팝업 스크린으로 값을 받기 때문)
*        plono     TYPE zc102ppt0006-plono,
        text(100),
      END OF gs_text,
      gt_text LIKE TABLE OF gs_text.


DATA : gv_from(1).   "조회용인지 작성용인지 구분용

*-- 텍스트 에디터를 위한 인덱스 저장
DATA : gv_tabix TYPE sy-tabix.


DATA: gv_empno_inserted.  " 글로벌 변수 선언 (사원 정보 유효성 체크)

DATA : gv_button_grid TYPE string.  " TAB1일때 tab1 Read, TAB3일때 tab3 Read, POP일때 pop Read, 반려 상세 사유 ALV 버튼 체크 용

DATA: gv_modified TYPE abap_bool.
