``` abap
*&---------------------------------------------------------------------*
*& Include ZC102SDR0004TOP                          - Report ZC102SDR0004
*&---------------------------------------------------------------------*
REPORT zc102sdr0004 MESSAGE-ID zc102msg.

**********************************************************************
*TABLES
**********************************************************************
TABLES: zc102sdt0004, zc102sdt0005, zc102mmt0001.


**********************************************************************
*call instance
**********************************************************************
DATA: go_container       TYPE REF TO cl_gui_custom_container,
      go_split_container TYPE REF TO cl_gui_splitter_container,

      go_left_container  TYPE REF TO cl_gui_container,
      go_right_container TYPE REF TO cl_gui_container,

      go_left_grid       TYPE REF TO cl_gui_alv_grid,
      go_right_grid      TYPE REF TO cl_gui_alv_grid,

*--item 팝업 띄우기
      go_pop_container   TYPE REF TO cl_gui_custom_container,
      go_pop_grid        TYPE REF TO cl_gui_alv_grid.



**********************************************************************
*itab and work area
**********************************************************************
*--input 값itab
DATA: BEGIN OF gs_input,
        vbeln_del TYPE zc102sdt0004-vbeln_del,
        partner   TYPE zc102sdt0004-partner,
      END OF gs_input,
      gt_input LIKE TABLE OF gs_input.

*--납품문서 헤더
*DATA: gt_del TYPE TABLE OF zc102sdt0004,
*      gs_del TYPE zc102sdt0004.

DATA: BEGIN OF gs_del.
        INCLUDE STRUCTURE zc102sdt0004.
DATA: d_state(10),
        ddtext(6),
      END OF gs_del,
      gt_del LIKE TABLE OF gs_del.

*-- 납품문서 테이블 배송상태 카운트 확인
DATA: BEGIN OF gs_iv,
        status TYPE icon-id.
        INCLUDE STRUCTURE zc102sdt0004.
DATA :  END OF gs_iv,
gt_iv LIKE TABLE OF gs_iv.

*--납품문서 서치헬프--*
DATA: BEGIN OF gs_vbeln,
        vbeln_del TYPE zc102sdt0004-vbeln_del,
      END OF gs_vbeln,
      gt_vbeln LIKE TABLE OF gs_vbeln.

*--partner 서치헬프--*
DATA: BEGIN OF gs_partner,
        partner   TYPE ZC102BPT0001-partner,
        name1  TYPE ZC102BPT0001-name1,
      END OF gs_partner,
      gt_partner LIKE TABLE OF gs_partner.

**배송상태 도메인 값**
DATA: gt_list TYPE TABLE OF dd07v,
      gs_list TYPE dd07v.


DATA : gs_do_h TYPE zc102sdt0004. " 판매 오더 - 납품 오더 조회시

*--납품문서 아이템
DATA: gt_del_item TYPE TABLE OF zc102sdt0005,
      gs_del_item TYPE zc102sdt0005.

*--자재관리
DATA: gt_mange TYPE TABLE OF zc102mmt0001,
      gs_mange TYPE zc102mmt0001.


*--for ALV
DATA: gs_lfcat        TYPE lvc_s_fcat,
      gs_rfcat        TYPE lvc_s_fcat,
      gs_pfcat        TYPE lvc_s_fcat,

      gt_lfcat        TYPE lvc_t_fcat,
      gt_rfcat        TYPE lvc_t_fcat,
      gt_pfcat        TYPE lvc_t_fcat,

      gs_layout_left  TYPE lvc_s_layo,
      gs_layout_right TYPE lvc_s_layo,
      gs_pop_layout   TYPE lvc_s_layo,

      gs_variant      TYPE disvariant,
      gs_variant_pop  TYPE disvariant.

*--toolbar
DATA: gt_ui_functions TYPE ui_functions,
      gs_button       TYPE stb_button,
      gs_button_pop   TYPE stb_button.

**********************************************************************
*variant
**********************************************************************
DATA: gv_okcode TYPE sy-ucomm.

*RANGES: gr_vbeln_del FOR zc102sdt0004-vbeln_del,
*        gr_partner   FOR zc102sdt0004-partner.

RANGES: gr_vbeln_del FOR gs_input-vbeln_del,
        gr_partner   FOR gs_input-partner.

DATA: gv_vbeln_del TYPE zc102sdt0004-vbeln_del,
      gv_partner   TYPE zc102sdt0004-partner.

DATA: gv_radio1(1),
      gv_radio2(1),
      gv_radio3(1).

DATA: gv_cnt1 TYPE i,
      gv_cnt2 TYPE i,
      gv_cnt3 TYPE i,
      gv_cnt4 TYPE i,
      gv_cnt5 TYPE i.


*-- FROM 판매오더 조회(BY 소연)
DATA : gv_from TYPE abap_bool VALUE abap_false.
