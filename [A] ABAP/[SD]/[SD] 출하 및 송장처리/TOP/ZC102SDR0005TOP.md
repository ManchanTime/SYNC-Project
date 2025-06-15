``` abap
*&---------------------------------------------------------------------*
*& Include ZC102SDR0005TOP                          - Report ZC102SDR0005
*&---------------------------------------------------------------------*
REPORT zc102sdr0005 MESSAGE-ID zc102msg.

**********************************************************************
* TYPE-POOLS
**********************************************************************
TYPE-POOLS ole2 .

**********************************************************************
* Macro
**********************************************************************
DEFINE _popup_to_confirm.

  CALL FUNCTION 'ZFM_CL102_CM_POP_UP'
    EXPORTING
      iv_titlebar       = &1
      iv_question       = &2
   IMPORTING
     ev_answer         =  &3.

END-OF-DEFINITION.

**********************************************************************
* Class Instance
**********************************************************************
DATA : go_container TYPE REF TO cl_gui_custom_container,
       go_alv_grid  TYPE REF TO cl_gui_alv_grid.

DATA : go_pop_cont TYPE REF TO cl_gui_custom_container,
       go_pop_grid TYPE REF TO cl_gui_alv_grid.

**********************************************************************
* Internal Table and Work Area
**********************************************************************
DATA : BEGIN OF gs_delivery,
         vbeln_del      TYPE zc102sdt0004-vbeln_del, "납품 번호
         vbeln_so       TYPE zc102sdt0004-vbeln_so,  "판매 오더 번호
         partner        TYPE zc102bpt0001-partner,   "BP 파트너
         dreal          TYPE zc102sdt0004-dreal,     "출하 요청일
         wadat_ist      TYPE zc102sdt0004-wadat_ist, "배송 시작일
         vsbed          TYPE zc102sdt0011-vsbed,     "배송 방법
         adrnr          TYPE zc102sdt0011-adrnr,     "배송지
         telf1          TYPE zc102sdt0001-telf1,     "배송 번호
         gbstk          TYPE zc102sdt0004-gbstk,     "배송상태
         state          TYPE zc102sdt0004-state,     "납품상태
         del_char       TYPE zc102sdt0011-del_char,  "배송 담당자
         vbeln_bil      TYPE zc102sdt0004-vbeln_bil, "송장 번호
         iseme          TYPE zc102sdt0004-iseme,     "납기
         icon           TYPE icon-id,                "납기일 임박
         color          TYPE lvc_t_scol,             "색
         modi_yn,                                    "1자리 수정
         deli_btn,                                   "삭제 버튼
         cell_tab       TYPE lvc_t_styl,             "필드 설정
         ltext          TYPE icon_d,                 "icon
         cusno          TYPE zc102sdt0004-cusno,     "고객번호
         finalsp        TYPE zc102sdt0004-finalsp,   "금액
         waers          TYPE zc102sdt0004-waers,     "통화키
         vdatu          TYPE zc102sdt0004-vdatu,     "납품예정일
         ddone          TYPE zc102sdt0004-ddone,     "납품완료일
         vsbed_name(10),                             "도메인 (배송 방법)
         gbstk_name(10),                             "도메인 (출하 상태)
       END OF gs_delivery,
       gt_delivery LIKE TABLE OF gs_delivery.

*-- 엑셀용 Internal Table
DATA : BEGIN OF gs_excel,
         stras     TYPE zc102sdt0001-stras,          "배송지
         telf1     TYPE zc102sdt0001-telf1,          "고객연락처
         matnr     TYPE zc102sdt0005-matnr,          "자재 명
         menge     TYPE zc102sdt0005-menge,          "수량
         meins     TYPE zc102sdt0005-meins,          "단위
         adrnr     TYPE zc102sdt0011-adrnr,          "주소지
         del_char  TYPE zc102sdt0011-del_char,       "배송담당자
         vsbed     TYPE zc102sdt0011-vsbed,          "배송 방법
         gbstk     TYPE zc102sdt0004-gbstk,          "출하 상태
         dreal     TYPE zc102sdt0004-dreal,          "실배송일
         vbeln_del TYPE zc102sdt0005-vbeln_del,      "납품 번호
       END OF gs_excel,
       gt_excel LIKE TABLE OF gs_excel.

*-- For Back-up
DATA : gt_backup LIKE TABLE OF gs_delivery,
       gs_backup LIKE gs_delivery.

*-- For 납품 ITEM
DATA : gs_item TYPE zc102sdt0005,
       gt_item TYPE TABLE OF zc102sdt0005.

DATA : gs_maktx TYPE zc102mmt0004,
       gt_maktx TYPE TABLE OF zc102mmt0004.

*-- For 판매오더
DATA : BEGIN OF gs_iorder.
         INCLUDE STRUCTURE zc102sdt0007.
DATA :   maktx TYPE zc102mmt0004-maktx,
       END OF gs_iorder,
       gt_iorder LIKE TABLE OF gs_iorder.

DATA: gt_horder TYPE TABLE OF zc102sdt0006,
      gs_horder TYPE zc102sdt0006.

*-- For ALV
DATA : gt_fcat    TYPE lvc_t_fcat,
       gs_fcat    TYPE lvc_s_fcat,
       gs_variant TYPE disvariant,
       gt_sort    TYPE lvc_t_sort,
       gs_sort    TYPE lvc_s_sort,
       gs_layout  TYPE lvc_s_layo.

DATA : gt_ui_functions TYPE ui_functions,
       gs_button       TYPE stb_button.

DATA : gt_pop_fcat TYPE lvc_t_fcat,
       gs_pop_fcat TYPE lvc_s_fcat,
       gs_playout  TYPE lvc_s_layo.

*-- For ALV List box
DATA : gt_drop TYPE lvc_t_drop,
       gs_drop TYPE lvc_s_drop.

**********************************************************************
* Common Variabe
**********************************************************************
DATA : gv_okcode TYPE sy-ucomm,
       gv_mode   VALUE 'D'.

DATA : gv_rest_cnt TYPE i, "남은 출고 건수
       gv_past_cnt TYPE i, "출고가 되지 않은 건수
       gv_requ_cnt TYPE i. "출고 완료 건수

DATA : gv_filter VALUE abap_true.

*-- For file browser
DATA : objfile       TYPE REF TO cl_gui_frontend_services,
       pickedfolder  TYPE string,
       initialfolder TYPE string,
       fullinfo      TYPE string,
       pfolder       TYPE rlgrap-filename. "MEMORY ID mfolder.

*-- For Excel
DATA: gv_tot_page   LIKE sy-pagno,          " Total page
      gv_percent(3) TYPE n,                 " Reading percent
      gv_file       LIKE rlgrap-filename .  " File name

DATA : gv_temp_filename     LIKE rlgrap-filename,
       gv_temp_filename_pdf LIKE rlgrap-filename,
       gv_form(40),
       gv_pdf.

DATA: excel       TYPE ole2_object,
      workbook    TYPE ole2_object,
      books       TYPE ole2_object,
      book        TYPE ole2_object,
      sheets      TYPE ole2_object,
      sheet       TYPE ole2_object,
      activesheet TYPE ole2_object,
      application TYPE ole2_object,
      pagesetup   TYPE ole2_object,
      cells       TYPE ole2_object,
      cell        TYPE ole2_object,
      row         TYPE ole2_object,
      buffer      TYPE ole2_object,
      font        TYPE ole2_object,
      range       TYPE ole2_object,  " Range
      borders     TYPE ole2_object.

DATA: cell1 TYPE ole2_object,
      cell2 TYPE ole2_object.

*-- 자동 채번 로직
DATA: gv_number    TYPE n LENGTH 10,     " 도메인의 길이에 맞게
      gv_prefix(3),  " PPO, PD0 등
      gv_full_code TYPE string,
      gv_range_nr  TYPE inri-nrrangenr,
      gv_quantity  TYPE inri-quantity.

DATA: ls_nriv TYPE nriv.

gv_prefix = 'I'.    " 원하는 prefix 사용
gv_range_nr = '01'. " 원하는 번호 - 도메인 정의서에 있음!!!
gv_quantity = 1.    " 원하는 증가량 사용
