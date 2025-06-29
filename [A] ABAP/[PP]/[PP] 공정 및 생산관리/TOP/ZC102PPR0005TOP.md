``` abap
*&---------------------------------------------------------------------*
*& Include ZC102PPR0005TOP                          - Report ZC102PPR0005
*&---------------------------------------------------------------------*
REPORT zc102ppr0005 MESSAGE-ID zc102msg.

**********************************************************************
*TABLES
**********************************************************************
TABLES: zc102ppt0012, zc102hrt0001,
        zc102ppt0002, zc102ppt0010, zc102ppt0011.

**********************************************************************
*tab strip
**********************************************************************
CONTROLS gc_tab TYPE TABSTRIP.

DATA: gv_subscreen TYPE sy-dynnr.

**********************************************************************
*call instance
**********************************************************************
DATA: go_container TYPE REF TO cl_gui_custom_container,
      go_alv_grid  TYPE REF TO cl_gui_alv_grid.

*--tab--*
DATA: go_container1 TYPE REF TO cl_gui_custom_container,
      go_alv_grid1  TYPE REF TO cl_gui_alv_grid,

      go_container2 TYPE REF TO cl_gui_custom_container,
      go_alv_grid2  TYPE REF TO cl_gui_alv_grid,

      go_container3 TYPE REF TO cl_gui_custom_container,
      go_alv_grid3  TYPE REF TO cl_gui_alv_grid,

      go_container4 TYPE REF TO cl_gui_custom_container,
      go_alv_grid4  TYPE REF TO cl_gui_alv_grid.


*--팝업 띄우기
*      go_pop_container    TYPE REF TO cl_gui_custom_container,
*      go_pop_grid         TYPE REF TO cl_gui_alv_grid,
*
*      go_pop_container2   TYPE REF TO cl_gui_custom_container,
*      go_pop_grid2        TYPE REF TO cl_gui_alv_grid.


**********************************************************************
*itab and work area
**********************************************************************
*DATA: BEGIN OF gs_input,
*        werks TYPE zc102ppt0012-werks,
*END OF gs_input,
*gt_input LIKE TABLE OF gs_input.

*--생산오더 alv 왼쪽--*
DATA: BEGIN OF gs_pdo,
        pdono       TYPE  zc102ppt0012-pdono,
        wkcno       TYPE  zc102ppt0012-wkcno,
        werks       TYPE zc102ppt0012-werks,
        rouno       TYPE zc102ppt0012-rouno,
        plono       TYPE zc102ppt0012-plono,
        pdstt       TYPE zc102ppt0012-pdstt,
        pdfns       TYPE zc102ppt0012-pdfns,
        mksta       TYPE zc102ppt0012-mksta,
        matnr       TYPE zc102ppt0012-matnr,
        menge       TYPE zc102ppt0012-menge,
        meins       TYPE zc102ppt0012-meins,
        prog        TYPE zc102ppt0012-prog,
        p_unit      TYPE zc102ppt0012-p_unit,
        d_state(10),
        ddtext(6),
      END OF gs_pdo,
      gt_pdo LIKE TABLE OF gs_pdo.

**진행상태 도메인 값**
DATA: gt_list TYPE TABLE OF dd07v,
      gs_list TYPE dd07v.

*--플랜트 번호 서치헬프--*
DATA: BEGIN OF gs_search,
        werks TYPE zc102ppt0012-werks,
        text  TYPE dd07v-ddtext, "도메인 텍스트 읽기용
      END OF gs_search,
      gt_search LIKE TABLE OF gs_search.

"도메인으로 관리되는 텍스트까지 불러올 수 있음 서치헬프 부분
DATA: gt_domval TYPE TABLE OF dd07v,
      gs_domval TYPE dd07v.

*--우측 tab 라우팅--*

"라우팅1
DATA: BEGIN OF gs_route,
        rouno  TYPE zc102ppt0010-rouno,
        wctno  TYPE zc102ppt0010-wctno,
        matnr  TYPE zc102ppt0010-matnr,
        pdono  TYPE zc102ppt0010-pdono,
        werks  TYPE zc102ppt0010-werks,
        rstda  TYPE zc102ppt0010-rstda,
        renda  TYPE zc102ppt0010-renda,
        reman  TYPE zc102ppt0010-reman,
        remac  TYPE zc102ppt0010-remac,
        status TYPE zc102ppt0010-status,
        menge  TYPE zc102ppt0010-menge,
        meins  TYPE zc102ppt0010-meins,
        prog   TYPE zc102ppt0010-prog,
        p_unit TYPE zc102ppt0012-p_unit,
        icon   TYPE icon-id,
      END OF  gs_route,
      gt_route LIKE TABLE OF gs_route.

"라우팅2
DATA: BEGIN OF gs_route2,
        rouno  TYPE zc102ppt0010-rouno,
        wctno  TYPE zc102ppt0010-wctno,
        matnr  TYPE zc102ppt0010-matnr,
        pdono  TYPE zc102ppt0010-pdono,
        werks  TYPE zc102ppt0010-werks,
        rstda  TYPE zc102ppt0010-rstda,
        renda  TYPE zc102ppt0010-renda,
        reman  TYPE zc102ppt0010-reman,
        remac  TYPE zc102ppt0010-remac,
        status TYPE zc102ppt0010-status,
        menge  TYPE zc102ppt0010-menge,
        meins  TYPE zc102ppt0010-meins,
        prog   TYPE zc102ppt0010-prog,
        p_unit TYPE zc102ppt0012-p_unit,
        icon   TYPE icon-id,
      END OF  gs_route2,
      gt_route2 LIKE TABLE OF gs_route2.

"라우팅3
DATA: BEGIN OF gs_route3,
        rouno  TYPE zc102ppt0010-rouno,
        wctno  TYPE zc102ppt0010-wctno,
        matnr  TYPE zc102ppt0010-matnr,
        pdono  TYPE zc102ppt0010-pdono,
        werks  TYPE zc102ppt0010-werks,
        rstda  TYPE zc102ppt0010-rstda,
        renda  TYPE zc102ppt0010-renda,
        reman  TYPE zc102ppt0010-reman,
        remac  TYPE zc102ppt0010-remac,
        status TYPE zc102ppt0010-status,
        menge  TYPE zc102ppt0010-menge,
        meins  TYPE zc102ppt0010-meins,
        prog   TYPE zc102ppt0010-prog,
        p_unit TYPE zc102ppt0012-p_unit,
        icon   TYPE icon-id,
      END OF  gs_route3,
      gt_route3 LIKE TABLE OF gs_route3.

"라우팅4
DATA: BEGIN OF gs_route4,
        rouno  TYPE zc102ppt0010-rouno,
        wctno  TYPE zc102ppt0010-wctno,
        matnr  TYPE zc102ppt0010-matnr,
        pdono  TYPE zc102ppt0010-pdono,
        werks  TYPE zc102ppt0010-werks,
        rstda  TYPE zc102ppt0010-rstda,
        renda  TYPE zc102ppt0010-renda,
        reman  TYPE zc102ppt0010-reman,
        remac  TYPE zc102ppt0010-remac,
        status TYPE zc102ppt0010-status,
        menge  TYPE zc102ppt0010-menge,
        meins  TYPE zc102ppt0010-meins,
        prog   TYPE zc102ppt0010-prog,
        p_unit TYPE zc102ppt0012-p_unit,
        icon   TYPE icon-id,
      END OF  gs_route4,
      gt_route4 LIKE TABLE OF gs_route4.

*--워크센터--*
DATA: BEGIN OF gs_work,
        wctno TYPE zc102ppt0011-wctno,
        werks TYPE zc102ppt0011-werks,
        manzt TYPE zc102ppt0011-manzt,
        pernr TYPE zc102ppt0011-pernr,
      END OF gs_work,
      gt_work LIKE TABLE OF gs_work.

**--자재마스터 테이블
*DATA: gt_master TYPE TABLE OF zc102mmt0004,
*      gs_master TYPE zc102mmt0004.

*--숙성창고--*
DATA: gt_ripen TYPE TABLE OF zc102mmt0014,
      gs_ripen TYPE zc102mmt0014.

*--for ALV
DATA: gt_left_fcat    TYPE lvc_t_fcat,
      gt_right_fcat   TYPE lvc_t_fcat,
      gt_pfcat        TYPE lvc_t_fcat,

      gs_left_fcat    TYPE lvc_s_fcat,
      gs_right_fcat   TYPE lvc_s_fcat,
      gs_pfcat        TYPE lvc_s_fcat,

      gs_layout       TYPE lvc_s_layo,
      gs_right_layout TYPE lvc_s_layo,
      gs_playout      TYPE lvc_s_layo,

      gs_variant      TYPE disvariant,
      gs_variant_pop  TYPE disvariant.

*--toolbar
DATA: gt_ui_functions TYPE ui_functions,
      gs_button       TYPE stb_button,
      gs_button_pop   TYPE stb_button.

**********************************************************************
*variant
**********************************************************************
DATA: gv_okcode TYPE sy-ucomm,
      gv_mode   VALUE 'E'.

DATA: gv_new_rouno TYPE zc102ppt0010-rouno,
      gv_today     TYPE sy-datum.

*--배치번호 채번--*
DATA: gv_bcno TYPE string.

DATA:gv_title1(20), " 텝 타이틀 변수 선언
     gv_title2(20),
     gv_title3(20),
     gv_title4(20).

DATA: gv_cnt1(3),
      gv_cnt2(3),
      gv_cnt3(3),
      gv_cnt4(3).   " 메세지 띄울때 총합 한번에 보려고 만듦
