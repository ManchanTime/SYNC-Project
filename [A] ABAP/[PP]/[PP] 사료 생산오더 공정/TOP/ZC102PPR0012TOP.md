``` abap
*&---------------------------------------------------------------------*
*& Include ZC102PPR0012TOP                          - Report ZC102PPR0012
*&---------------------------------------------------------------------*
REPORT zc102ppr0012 MESSAGE-ID zc102msg.

**********************************************************************
*TABLES
**********************************************************************
TABLES : zc102ppt0014.

**********************************************************************
*tab strip
**********************************************************************
CONTROLS gc_tab TYPE TABSTRIP.

DATA: gv_subscreen TYPE sy-dynnr.
**********************************************************************
*Class instance
**********************************************************************
DATA: go_container TYPE REF TO cl_gui_custom_container,        "사료 생산오더 현황
      go_alv_grid  TYPE REF TO cl_gui_alv_grid.

*--Tab
DATA: go_container1 TYPE REF TO cl_gui_custom_container,
      go_alv_grid1  TYPE REF TO cl_gui_alv_grid,

      go_container2 TYPE REF TO cl_gui_custom_container,
      go_alv_grid2  TYPE REF TO cl_gui_alv_grid,

      go_container3 TYPE REF TO cl_gui_custom_container,
      go_alv_grid3  TYPE REF TO cl_gui_alv_grid.

**********************************************************************
*Internal table and  Workarea
**********************************************************************
" 위 ALV
DATA : BEGIN OF gs_fo.                    "생산오더 인터널 테이블
         INCLUDE STRUCTURE zc102ppt0014.
DATA :  END OF gs_fo,
gt_fo LIKE TABLE OF gs_fo.


" 아래 ALV
DATA : BEGIN OF gs_fo1.                    "라우팅 1 분쇄
         INCLUDE STRUCTURE zc102ppt0014.
DATA :  END OF gs_fo1,
gt_fo1 LIKE TABLE OF gs_fo1.

DATA : BEGIN OF gs_fo2.                    "라우팅 2 건조
         INCLUDE STRUCTURE zc102ppt0014.
DATA :  END OF gs_fo2,
gt_fo2 LIKE TABLE OF gs_fo2.


DATA : BEGIN OF gs_fo3.                    "라우팅 3 포장
         INCLUDE STRUCTURE zc102ppt0014.
DATA :  END OF gs_fo3,
gt_fo3 LIKE TABLE OF gs_fo3.

*--for ALV
DATA: gt_fcat_1   TYPE lvc_t_fcat,
      gt_fcat_2   TYPE lvc_t_fcat,

      gs_fcat_1   TYPE lvc_s_fcat,
      gs_fcat_2   TYPE lvc_s_fcat,

      gs_layout_1 TYPE lvc_s_layo,
      gs_layout_2 TYPE lvc_s_layo,

      gs_variant  TYPE disvariant.

*--toolbar
DATA: gt_ui_functions TYPE ui_functions.
**********************************************************************
*Common value
**********************************************************************
DATA: gv_okcode TYPE sy-ucomm.

DATA: gv_title1(20), " 텝 타이틀 변수 선언
      gv_title2(20),
      gv_title3(20),

      gv_cnt1(3),
      gv_cnt2(3),
      gv_cnt3(3),
       gv_cnt_total(3).
