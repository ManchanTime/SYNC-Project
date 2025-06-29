``` abap
*&---------------------------------------------------------------------*
*& Include ZC102PPR0011TOP                          - Report ZC102PPR0011
*&---------------------------------------------------------------------*
REPORT zc102ppr0011 MESSAGE-ID zc102msg.


**********************************************************************
*Tap Strip controls
**********************************************************************
CONTROLS gc_tab TYPE TABSTRIP.      " TAB Strip object

DATA : gv_subscreen TYPE sy-dynnr.  " Subscreen number

**********************************************************************
* CLASS INSTANCE
**********************************************************************
DATA : go_container TYPE REF TO cl_gui_custom_container,
       go_alv_grid  TYPE REF TO cl_gui_alv_grid.

DATA : go_tab_cont1 TYPE REF TO cl_gui_custom_container,
       go_tab_grid1 TYPE REF TO cl_gui_alv_grid,
       go_tab_cont2 TYPE REF TO cl_gui_custom_container,
       go_tab_grid2 TYPE REF TO cl_gui_alv_grid,
       go_tab_cont3 TYPE REF TO cl_gui_custom_container,
       go_tab_grid3 TYPE REF TO cl_gui_alv_grid.

*-- Top-of-page
DATA : go_top_cont   TYPE REF TO cl_gui_docking_container,
       go_dyndoc_id  TYPE REF TO cl_dd_document,
       go_html_cntrl TYPE REF TO cl_gui_html_viewer.

**********************************************************************
* Internal table and Work area
**********************************************************************
*-- 사료 자재 창고
DATA : BEGIN OF gs_storage.
         INCLUDE STRUCTURE zc102mmt0005.
DATA :   maktx TYPE zc102mmt0004-maktx,
       END OF gs_storage,
       gt_storage LIKE TABLE OF gs_storage.


*-- 생산 오더
DATA : gt_po_not  TYPE TABLE OF zc102ppt0014, " 생산 전 PO
       gt_po_ing  TYPE TABLE OF zc102ppt0014, " 생산 중 PO
       gt_po_done TYPE TABLE OF zc102ppt0014, " 생산 후 PO
       gs_po      TYPE zc102ppt0014.          " 생산오더 WA

*-- BOM 아이템
DATA : BEGIN OF gs_bom,
         bom_h TYPE zc102ppt0004-bom_h, " BOM 헤더 넘버
         bomno TYPE zc102ppt0004-bomno, " BOM 아이템 넘버
         matnr TYPE zc102ppt0004-matnr, " 자재번호
         menge TYPE zc102ppt0004-menge, " 필요수량
         meins TYPE zc102ppt0004-meins, " 단위
         nowme TYPE zc102ppt0004-menge, " 현재 수량
         okmen TYPE zc102ppt0004-menge, " 생산 가능 수량
         disme TYPE zc102mmt0018-disme, " 남은(폐기할) 수량
       END OF gs_bom,
       gt_bom LIKE TABLE OF gs_bom.

*-- For ALV
DATA : gt_fcat         TYPE lvc_t_fcat, " TOP Field catalog
       gs_fcat         TYPE lvc_s_fcat,
       gs_layout       TYPE lvc_s_layo,
       gs_variant      TYPE disvariant,

       gt_tab_fcat1    TYPE lvc_t_fcat,   " 탭 1 FCAT
       gs_tab_fcat1    TYPE lvc_s_fcat,

       gt_tab_fcat2    TYPE lvc_t_fcat,   " 탭 2 FCAT
       gs_tab_fcat2    TYPE lvc_s_fcat,

       gt_tab_fcat3    TYPE lvc_t_fcat,   " 탭 3 FCAT
       gs_tab_fcat3    TYPE lvc_s_fcat,

       gs_tab_variant1 TYPE disvariant,
       gs_tab_variant2 TYPE disvariant,
       gs_tab_variant3 TYPE disvariant,
       gs_tab_layout   TYPE lvc_s_layo.

DATA : gt_ui_functions TYPE ui_functions,
       gs_button       TYPE stb_button.

DATA : gs_return TYPE zc102mmt0018,          " 폐기 WA
       gt_return TYPE TABLE OF zc102mmt0018. " 폐기 IT

*-- 자재별 SUM
DATA : gs_sort TYPE lvc_s_sort,
       gt_sort TYPE lvc_t_sort.

**********************************************************************
* Common variable
**********************************************************************
DATA : gv_okcode TYPE sy-ucomm.

*-- Screen 100
DATA : gv_matnr_c TYPE zc102mmt0004-matnr, " 닭 전용 사료 자재번호
       gv_matnr_p TYPE zc102mmt0004-matnr, " 돼지 전용 사료 자재번호
       gv_chi     TYPE zc102ppt0012-menge, " 닭 사료 생산 가능 수량
       gv_pig     TYPE zc102ppt0012-menge. " 돼지 사료 생산 가능 수량

DATA : gv_no_cnt(20),
       gv_ing_cnt(20),
       gv_done_cnt(20),
       gv_n(3),
       gv_i(3),
       gv_d(3).

*-- 사료 플랜트, 창고번호
DATA : gv_werks TYPE zc102mmt0008-werks,
       gv_stlno TYPE zc102mmt0008-stlno.

*-- 사료 BOM 넘버
DATA : gv_bom_c TYPE zc102ppt0003-bomno, " 닭 사료 BOM 번호
       gv_bom_p TYPE zc102ppt0003-bomno. " 돼지 사료 BOM 번호

*-- 사료 단위
DATA : gv_cunit TYPE zc102ppt0014-meins,
       gv_punit TYPE zc102ppt0014-meins.

*-- 폐기리스트 번호
DATA : gv_retno TYPE zc102mmt0018-disno.

*--생산오더 번호 채번
DATA : gv_pdono TYPE zc102ppt0014-pdono.
