``` abap
*&---------------------------------------------------------------------*
*& Include ZC102PPR0002TOP                          - Report ZC102PPR0002
*&---------------------------------------------------------------------*
REPORT zc102ppr0002 MESSAGE-ID zc102msg.


**********************************************************************
*TABLES
**********************************************************************
TABLES: zc102ppt0004, zc102sdt0007, zc102ppt0011, zc102sdt0006,
        zc102ppt0006, zc102mmt0001.


**********************************************************************
*call instance
**********************************************************************
DATA: go_container              TYPE REF TO cl_gui_custom_container, "상단 메인
      go_split_container        TYPE REF TO cl_gui_splitter_container, "상단 스플릿
      go_right_split_container  TYPE REF TO cl_gui_splitter_container,

      go_bottom_split_container TYPE REF TO cl_gui_splitter_container, "하단 스플릿

      go_top_container          TYPE REF TO cl_gui_container,
      go_right_container        TYPE REF TO cl_gui_container,
      go_bottom_container       TYPE REF TO cl_gui_container,
      go_bottom2_container      TYPE REF TO cl_gui_container,

      go_top_grid               TYPE REF TO cl_gui_alv_grid,
      go_bottom_grid            TYPE REF TO cl_gui_alv_grid,
      go_bottom2_grid           TYPE REF TO cl_gui_alv_grid,

*--item 팝업 띄우기
      go_pop_container          TYPE REF TO cl_gui_custom_container,
      go_split_container_pop    TYPE REF TO cl_gui_splitter_container,

      go_ptop_container         TYPE REF TO cl_gui_container,
      go_pbottom_container      TYPE REF TO cl_gui_container,

      go_ptop_grid              TYPE REF TO cl_gui_alv_grid,
      go_pbottom_grid           TYPE REF TO cl_gui_alv_grid,

      go_pop_container2         TYPE REF TO cl_gui_docking_container,
      go_pr_pop_grid            TYPE REF TO cl_gui_alv_grid.


**********************************************************************
*itab and work area
**********************************************************************
*--BP 주문 정보--*
DATA: gs_bpinfo TYPE zc102sdt0010,
      gt_bpinfo TYPE TABLE OF zc102sdt0010.

*--BP마스터--*
DATA: gs_bpmarster TYPE zc102mmt0019,
      gt_bpmarster TYPE TABLE OF zc102mmt0019.

*--긴급판매 오더 아이템 가져오기--*
DATA: BEGIN OF gs_item.
        INCLUDE STRUCTURE zc102sdt0007.

DATA:   plono  TYPE zc102ppt0006-plono, "생산계획번호
        maktx  TYPE zc102mmt0004-maktx, "자재명
        ortype TYPE zc102sdt0006-ortype, "오더 유형
        mrp_st TYPE zc102sdt0006-mrp_st, "mrp상태
        delid  TYPE zc102sdt0006-delid, "납품일
        audat  TYPE zc102sdt0006-audat, "주문 일자
      END   OF   gs_item,
      gt_item LIKE TABLE OF gs_item.

DATA: gv_vbeln TYPE zc102sdt0007-vbeln_so.

*--계획오더 생성 완료 부분--*
DATA: BEGIN OF gs_planto,
        plono    TYPE zc102ppt0006-plono, "생산계획번호
        partner  TYPE zc102sdt0007-partner, "BP번호
        matnr    TYPE zc102sdt0007-matnr, "자재번호
        maktx    TYPE zc102mmt0004-maktx, "자재명
        pequan   TYPE zc102sdt0007-pequan, "생산 수량
        meins    TYPE zc102sdt0007-meins, "단위
        stlno    TYPE zc102ppt0006-stlno, "창고번호
        ref_no   TYPE zc102sdt0007-vbeln_so, "판매오더번호
        psttr    TYPE zc102ppt0006-psttr, "계획 생산일
        menge    TYPE zc102ppt0006-menge, "수량
        pedtr    TYPE zc102ppt0006-pedtr, "계획 종료일
        werks    TYPE zc102ppt0006-werks, "플랜트 번호
        wkcno    TYPE zc102ppt0006-wkcno, "공장번호
        mtype    TYPE zc102sdt0010-mtype, "자재 유형
        erdat    TYPE zc102ppt0006-erdat, "생성일
        ernam    TYPE zc102ppt0006-ernam, "생성자
        erzet    TYPE zc102ppt0006-erzet, "생성시간
        cell_tab TYPE lvc_t_styl,
        status   TYPE zc102ppt0006-status,
        modi_yn,
      END OF gs_planto,
      gt_planto LIKE TABLE OF gs_planto.

DATA: gt_planto_s TYPE TABLE OF zc102ppt0006.

*--구매요청 생성완료 부분--*
DATA: BEGIN OF gs_prto,
        prno  TYPE zc102mmt0016-prno, "구매요청번호
        menge TYPE zc102mmt0016-menge, "수량
        tolco TYPE zc102mmt0016-tolco, "구매 총액
        waers TYPE zc102mmt0016-waers, "통화키
        redat TYPE zc102mmt0016-redat, "요청일자
      END OF gs_prto,
      gt_prto LIKE TABLE OF gs_prto.

*--자재관리에서 자재 계산에서 가져오기--*
DATA: BEGIN OF gs_inidivi_bom,
        matnr    TYPE zc102sdt0007-matnr, "자재명
        stock    TYPE zc102mmt0001-labst, "가용재고량
        meins    TYPE zc102mmt0001-meins, "단위
        shortage TYPE i, "부족수량
      END OF gs_inidivi_bom,
      gt_inidivi_bom LIKE TABLE OF gs_inidivi_bom.

*--차감할 가용재고량과 창고번호 담을 itab--*
DATA: BEGIN OF gs_minus,
        vbeln_so TYPE zc102sdt0007-vbeln_so, "판매오더번호
        matnr    TYPE zc102sdt0007-matnr, "자재번호
        stlno    TYPE zc102mmt0001-stlno, "창고번호
        werks    TYPE zc102mmt0001-werks, "플랜트 번호
        labst    TYPE zc102mmt0001-labst, "가용재고량
        menge    TYPE zc102ppt0004-menge, "완제품 생산에 필요한 수량
        needed   TYPE zc102ppt0004-menge, "주문받은 수량만큼 생산에 필요한 수량
        ppquan   TYPE zc102ppt0004-menge, "생산 가능수량
        lequan   TYPE zc102ppt0004-menge, "부족수량
        i_lequan TYPE zc102ppt0004-menge, "부족 수량이 있을 시 각각 bom자재 몇개 씩 부족한지
      END OF gs_minus,
      gt_minus LIKE TABLE OF gs_minus.


DATA: gt_mange_update TYPE TABLE OF zc102mmt0001,
      gs_mange_update TYPE zc102mmt0001.


********더블클릭 시 생산가능수량 부족수량 보여주기 위한***********
DATA: BEGIN OF gs_tlist,
        vbeln_so TYPE zc102sdt0007-vbeln_so, "판매오더번호
        matnr    TYPE zc102sdt0007-matnr, "자재번호
        maktx    TYPE zc102mmt0004-maktx, "자재명
        mtype    TYPE zc102mmt0004-mtype, "자재 유형
        pequan   TYPE zc102sdt0007-pequan, "요구수량
        meins    TYPE zc102sdt0007-meins, "단위
        werks    TYPE zc102ppt0006-werks, "플랜트번호
        wkcno    TYPE zc102ppt0006-wkcno, "공장 번호
        lequan   TYPE MENGE_d, "부족수량
        ppquan   TYPE MENGE_d, "생산가능수량
        status   TYPE icon-id, "mrp상태
        modi_yn,
      END OF  gs_tlist,
      gt_tlist LIKE TABLE OF gs_tlist.

*******생산계획 테이블에 옮기기 전 담아둘 곳*********
DATA: BEGIN OF gs_create_pdo,
        vbeln_so TYPE zc102sdt0007-vbeln_so, "판매오더 번호
        plono    TYPE zc102ppt0006-plono, "생산계획번호
        matnr    TYPE zc102sdt0007-matnr, "
        maktx    TYPE zc102mmt0004-maktx, "자재명
        ppquan   TYPE MENGE_d, "생산수량
        meins    TYPE zc102sdt0007-meins, "단위
        mtype    TYPE zc102mmt0004-mtype, "자재 유형
        pequan   TYPE zc102sdt0007-pequan, "요구수량
      END OF gs_create_pdo,
      gt_create_pdo LIKE TABLE OF gs_create_pdo.


*--BOM 테이블 가져오기--*
DATA: gt_bom TYPE TABLE OF zc102ppt0004,
      gs_bom TYPE zc102ppt0004.

*--자재관리 테이블 가져오기--*
DATA: gt_mange TYPE TABLE OF zc102mmt0001,
      gs_mange TYPE zc102mmt0001.

*--계획오더 테이블 가져오기--*
DATA: gt_plo TYPE TABLE OF zc102ppt0006,
      gs_plo TYPE zc102ppt0006.

*--판매오더번호 서치헬프--*
DATA: BEGIN OF gs_search,
        vbeln_so TYPE zc102sdt0007-vbeln_so,
      END OF gs_search,
      gt_search LIKE TABLE OF gs_search.

DATA: BEGIN OF gs_vbeln,
        vbeln_so TYPE zc102sdt0007-vbeln_so,
      END OF gs_vbeln,
      gt_vbeln LIKE TABLE OF gs_vbeln.

********리스트 하단 부분***********
DATA: BEGIN OF gs_blist,
        matnr   TYPE zc102sdt0007-matnr,
        maktx   TYPE zc102mmt0004-maktx,
        menge   TYPE zc102ppt0004-menge,
        labst   TYPE zc102mmt0001-labst,
        meins   TYPE zc102ppt0004-meins,
        lequan  TYPE MENGE_d,
        modi_yn,
      END OF  gs_blist,
      gt_blist LIKE TABLE OF gs_blist.

*--구매요청 헤더, 아이템 테이블
DATA: gt_hpr TYPE TABLE OF zc102mmt0016,
      gs_hpr TYPE zc102mmt0016,

      gt_ipr TYPE TABLE OF zc102mmt0017,
      gs_ipr TYPE zc102mmt0017.

*--자재마스터 테이블
DATA: gt_master TYPE TABLE OF zc102mmt0004,
      gs_master TYPE zc102mmt0004.

*--for ALV
DATA: gt_tfcat             TYPE lvc_t_fcat,
      gt_bfcat             TYPE lvc_t_fcat,
      gt_b2fcat            TYPE lvc_t_fcat,
      gt_ptfcat            TYPE lvc_t_fcat,
      gt_pbfcat            TYPE lvc_t_fcat,
      gt_pop2_fcat         TYPE lvc_t_fcat,

      gs_tfcat             TYPE lvc_s_fcat,
      gs_bfcat             TYPE lvc_s_fcat,
      gs_b2fcat            TYPE lvc_s_fcat,

      gs_ptfcat            TYPE lvc_s_fcat,
      gs_pbfcat            TYPE lvc_s_fcat,
      gs_pb2fcat           TYPE lvc_s_fcat,
      gs_pop2_fcat         TYPE lvc_s_fcat,

      gs_layout            TYPE lvc_s_layo,
      gs_layout_bottom     TYPE lvc_s_layo,
      gs_layout_bottom2    TYPE lvc_s_layo,

      gs_pop_layout        TYPE lvc_s_layo,
      gs_pop_layout_bottom TYPE lvc_s_layo,

      gs_variant           TYPE disvariant,
      gs_variant_pop       TYPE disvariant,
      gs_variant_pop2      TYPE disvariant.



*--toolbar
DATA: gt_ui_functions TYPE ui_functions,
      gs_button       TYPE stb_button,
      gs_button_pop   TYPE stb_button.

**********************************************************************
*variant
**********************************************************************
DATA: gv_okcode  TYPE sy-ucomm,
      gv_mode    VALUE 'P',
      gv_enabled.

DATA: gv_pono TYPE string, "계획오더 채번
      gv_prno TYPE string. "구매요청 채번


RANGES: gr_vbeln FOR zc102sdt0007-vbeln_so.
