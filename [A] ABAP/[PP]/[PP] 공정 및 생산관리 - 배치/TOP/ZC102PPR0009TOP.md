``` abap
*&---------------------------------------------------------------------*
*& Include ZC102PPR0009TOP                          - Report ZC102PPR0009
*&---------------------------------------------------------------------*
REPORT zc102ppr0009 MESSAGE-ID zc102msg.

**********************************************************************
*TABLES
**********************************************************************
TABLES: zc102ppt0010, zc102mmt0014, zc102ppt0012, zc102ppt0007.

**********************************************************************
*itab and work area
**********************************************************************
*--라우팅 마스터--*
DATA: gt_rmaster TYPE TABLE OF zc102ppt0009,
      gs_rmaster TYPE zc102ppt0009.

*--라우팅--*
DATA: gt_route TYPE TABLE OF zc102ppt0010,
      gs_route TYPE zc102ppt0010.

*--숙성창고--*
DATA: gt_ripen TYPE TABLE OF zc102mmt0014,
      gs_ripen TYPE zc102mmt0014.

*--생산오더--*
DATA: gt_pdo TYPE TABLE OF zc102ppt0012,
      gs_pdo TYPE zc102ppt0012.

*--생산완료 수정여부 확인용--*
DATA: gt_check TYPE TABLE OF zc102ppt0012,
      gs_check TYPE zc102ppt0012.

*--생산완료--*
DATA: gt_complit TYPE TABLE OF zc102ppt0007,
      gs_complit TYPE zc102ppt0007.


**********************************************************************
*variant
**********************************************************************
DATA: gv_new_rouno TYPE zc102ppt0010-rouno,
      gv_today     TYPE sy-datum.

*--배치번호 채번--*
DATA: gv_bcno TYPE string.
