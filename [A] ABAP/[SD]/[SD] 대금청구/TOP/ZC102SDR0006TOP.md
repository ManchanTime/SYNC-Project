``` abap
*&---------------------------------------------------------------------*
*& Include ZC102SDR0006TOP                          - Report ZC102SDR0006
*&---------------------------------------------------------------------*
REPORT zc102sdr0006 MESSAGE-ID zc102msg.

**********************************************************************
* TABLES
**********************************************************************
TABLES : zc102sdt0004, zc102sdt0005, " 납품 Header, Item
         zc102sdt0001.

**********************************************************************
* CLASS INSTANCE
**********************************************************************
DATA : go_container TYPE REF TO cl_gui_custom_container,
       go_alv_grid  TYPE REF TO cl_gui_alv_grid.

*-- For Popup
DATA : go_pop_cont TYPE REF TO cl_gui_custom_container,
       go_pop_grid TYPE REF TO cl_gui_alv_grid.

*-- Text editor용
DATA : go_text_edit TYPE REF TO cl_gui_textedit,
       go_text_cont TYPE REF TO cl_gui_custom_container.

**********************************************************************
* Internal table and Work area
**********************************************************************
*-- 납품 Header
DATA : BEGIN OF gs_header.
         INCLUDE STRUCTURE zc102sdt0004.
DATA :   vsbed_name(10),
         gbstk_name(10),
         dismo          TYPE zc102sdt0006-dismo,    " 할인가
         netwr          TYPE zc102sdt0006-netwr,    " 판매가
         stax           TYPE zc102sdt0006-stax,     " 부가세
         name1          TYPE zc102sdt0001-name1,
         leati          TYPE zc102sdt0011-leati,
         del_char       TYPE zc102sdt0011-del_char,
         status         TYPE icon-id,
         icon           TYPE icon_d,
         cell_tab       TYPE lvc_t_styl,
       END OF gs_header,
       gt_header LIKE TABLE OF gs_header.

*-- 납품 Line
DATA : BEGIN OF gs_line.
         INCLUDE STRUCTURE zc102sdt0005.
DATA :   tdline TYPE TABLE OF tline,
       END OF gs_line,
       gt_line LIKE TABLE OF gs_line.

*-- 대금청구
DATA : BEGIN OF gs_billing.
         INCLUDE STRUCTURE zc102sdt0009.
DATA : END OF gs_billing,
gt_billing LIKE TABLE OF gs_billing.

*-- For Search help
DATA : BEGIN OF gs_vbeln,
         vbeln_del TYPE zc102sdt0004-vbeln_del,
       END OF gs_vbeln,
       gt_vbeln LIKE TABLE OF gs_vbeln.

*-- For ALV
DATA : gs_fcat    TYPE lvc_s_fcat,
       gt_fcat    TYPE lvc_t_fcat,
       gs_layout  TYPE lvc_s_layo,
       gs_variant TYPE disvariant.

*-- For Popup ALV
DATA : gs_pfcat   TYPE lvc_s_fcat,
       gt_pfcat   TYPE lvc_t_fcat,
       gs_playout TYPE lvc_s_layo.

**********************************************************************
* Global variable
**********************************************************************
DATA : gv_okcode TYPE sy-ucomm,
       gv_email  TYPE zc102sdt0001-email. " 이메일 주소

RANGES : gr_vbeln_del FOR zc102sdt0004-vbeln_del,
         gr_partner FOR zc102sdt0004-partner,
         gr_wadat_ist FOR zc102sdt0004-wadat_ist.

DATA gt_editor_lines TYPE TABLE OF tline.
