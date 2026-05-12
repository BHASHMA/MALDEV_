

SEH overflow is a type of buffer overflow where we overwrite the Exception Registration Record stored on the stack. 
When a crash occurs, Windows uses Structured Exception Handling to walk the SEH chain starting from the TEB. 
Since we control the overwritten handler, execution is redirected to our controlled address, allowing us to hijack execution flow.


Overflow → overwrite SEH → trigger crash → Windows calls our handler → we control execution