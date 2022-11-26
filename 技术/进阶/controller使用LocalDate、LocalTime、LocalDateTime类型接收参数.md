```java
@RestController
@RequestMapping("/test")
public class TestController {
    @GetMapping("/date")
    public LocalDate showDate(@RequestParam @DateTimeFormat(iso = DateTimeFormat.ISO.DATE) LocalDate date) {
        return date;
    }


    @GetMapping("/time")
    public LocalTime showTime(@RequestParam @DateTimeFormat(pattern = "HH:mm:ss") LocalTime time) {
        return time;
    }
    @GetMapping("/dateTime")
    public LocalDateTime showDateTime(@RequestParam @DateTimeFormat(pattern = "yyyy-MM-dd HH:mm:ss") LocalDateTime dateTime) {
        return dateTime;
    }
}
```

[Spring MVC 接收 LocalDate、LocalTime 和 LocalDateTime Java 8 时间类型参数](https://www.cnblogs.com/qifenghao/p/12240943.html)