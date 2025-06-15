``` abap
*&---------------------------------------------------------------------*
*& Include ZC102PPR0013TOP                          - Report ZC102PPR0013
*&---------------------------------------------------------------------*
REPORT zc102ppr0013  MESSAGE-ID zc102msg.

**********************************************************************
TABLES: zc102ppt0010 ,zc102ppt0014, zc102ppt0007.

**********************************************************************
*Internal table and workarea
**********************************************************************
*--사료 생산오더
DATA : gt_fo TYPE TABLE OF zc102ppt0014,
       gs_fo TYPE zc102ppt0014.

*--사료 생산완료
DATA: gt_complit TYPE TABLE OF zc102ppt0007,
      gs_complit TYPE zc102ppt0007.

*--가공 완제품 테이블
DATA : gt_finish TYPE TABLE OF zc102mmt0003,
       gs_finish TYPE zc102mmt0003.

*--배치 번호 insert
DATA: gv_batno TYPE zc102ppt0005-batno,
      gs_batch TYPE zc102ppt0005,
      gt_batch TYPE TABLE OF zc102ppt0005.

**********************************************************************
*Varient
**********************************************************************
DATA:  gv_today TYPE sy-datum.
