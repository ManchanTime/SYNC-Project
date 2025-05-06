``` abap
PROCESS BEFORE OUTPUT.
  MODULE status_0200.
  MODULE init_process_control_0200.
*
PROCESS AFTER INPUT.
  MODULE user_command_0200.
  MODULE exit_0200 AT  EXIT-COMMAND.
