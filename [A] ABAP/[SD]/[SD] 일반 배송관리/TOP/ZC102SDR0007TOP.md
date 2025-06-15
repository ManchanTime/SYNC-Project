``` abap
*&---------------------------------------------------------------------*
*& Include ZC102SDR0007TOP                          - Report ZC102SDR0007
*&---------------------------------------------------------------------*
REPORT zc102sdr0007 MESSAGE-ID zc102msg.

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
TABLES : zc102sdt0004, zc102sdt0011.

**********************************************************************
* Class instance
**********************************************************************
DATA: go_main_cont  TYPE REF TO cl_gui_custom_container,    " 메인
      go_split_cont TYPE REF TO cl_gui_splitter_container,  " 좌우 분할
      go_left_cont  TYPE REF TO cl_gui_container,           " 좌
      go_right_cont TYPE REF TO cl_gui_container.           " 우

DATA : go_left_grid  TYPE REF TO cl_gui_alv_grid,         " -> 좌측 ALV
       go_right_grid TYPE REF TO cl_gui_alv_grid.         " -> 상단 ALV

**********************************************************************
* Internal table and Work area
**********************************************************************
*-- LEFT : 배송정보 미입력
DATA : BEGIN OF gs_left.
         INCLUDE STRUCTURE zc102sdt0004.
DATA :   adrnr          TYPE zc102sdt0011-adrnr,
         del_char       TYPE zc102sdt0011-del_char,
         vsbed          TYPE zc102sdt0011-vsbed,
         telf1          TYPE zc102sdt0011-telf1,
         leati          TYPE zc102sdt0011-leati,
         vsbed_name(10),
         gbstk_name(10),
       END OF gs_left,
       gt_left LIKE TABLE OF gs_left.

DATA : gs_fcat_right   TYPE lvc_s_fcat,
       gt_fcat_right   TYPE lvc_t_fcat,
       gs_layout_right TYPE lvc_s_layo,
       gs_variant      TYPE disvariant,
       gs_button       TYPE stb_button.

*-- RIGHT : 배송정보 입력완료
DATA : BEGIN OF gs_right.
         INCLUDE STRUCTURE zc102sdt0004.
DATA :   adrnr          TYPE zc102sdt0011-adrnr,
         del_char       TYPE zc102sdt0011-del_char,
         vsbed          TYPE zc102sdt0011-vsbed,
         telf1          TYPE zc102sdt0011-telf1,
         leati          TYPE zc102sdt0011-leati,
         vsbed_name(10),
         gbstk_name(10),
         modi_yn,
       END OF gs_right,
       gt_right LIKE TABLE OF gs_right.

DATA : gs_fcat_left   TYPE lvc_s_fcat,
       gt_fcat_left   TYPE lvc_t_fcat,
       gs_layout_left TYPE lvc_s_layo.

*-- 배송정보
DATA : BEGIN OF gs_deliv,
         name1 TYPE zc102bpt0001-name1,
         land1 TYPE zc102bpt0001-land1,
         stras TYPE zc102sdt0001-stras,
         telf1 TYPE zc102sdt0001-telf1,
       END OF gs_deliv.

*-- For Search help
DATA : BEGIN OF gs_vbeln,
         vbeln_del TYPE zc102sdt0004-vbeln_del,
       END OF gs_vbeln,
       gt_vbeln LIKE TABLE OF gs_vbeln.

DATA : BEGIN OF gs_sh_vsbed,
         vsbed      TYPE zc102sdt0011-vsbed,
         vsbed_name,
       END OF gs_sh_vsbed,
       gt_sh_vsbed LIKE TABLE OF gs_sh_vsbed.

**********************************************************************
* Global variable
**********************************************************************
DATA : gv_okcode TYPE sy-ucomm,
       gv_check  TYPE abap_bool,
       gv_mode   VALUE 'D'.

*-- 배송정보
DATA : gv_name1 TYPE zc102bpt0001-name1,
       gv_land1 TYPE zc102bpt0001-land1,
       gv_stras TYPE zc102sdt0001-stras,
       gv_telf1 TYPE zc102sdt0001-telf1.

RANGES : gr_vbeln_del FOR zc102sdt0004-vbeln_del.
