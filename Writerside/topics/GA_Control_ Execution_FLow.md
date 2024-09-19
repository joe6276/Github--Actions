# Control Execution Flow

At this point, we have jobs executed in a series, and if one fails, the rest are not executed. That's the default behavior.
![Execution](image_27.png)

But sometimes you want to keep executing even if one previous step fails, or even trigger another step if it fails or even control your execution flow.

## Conditional Jobs & steps

![Control Flow](image_28.png)