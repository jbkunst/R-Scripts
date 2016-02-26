# 
Joshua Kunst  



## Text intro


```r
url <- "http://www.gsmarena.com"

tabletd <- file.path(url, "makers.php3") %>% 
  read_html() %>% 
  html_nodes("table td")

dfbrands <- data_frame(
  td1 = tabletd[seq(1, length(tabletd), 2)],
  td2 = tabletd[seq(2, length(tabletd), 2)]
  ) %>%  
  mutate(brand_name = html_node(td2, "a") %>% html_text(),
         brand_url = html_node(td1, "a") %>% html_attr("href"),
         brand_image_url = html_node(td1, "img") %>% html_attr("src"),
         brand_n_phn = str_extract(brand_name, "\\(\\d+\\)"),
         brand_n_phn = str_replace_all(brand_n_phn, "\\(|\\)", ""),
         brand_n_phn = as.numeric(brand_n_phn),
         brand_name = str_replace_all(brand_name, " phones \\(\\d+\\)", "")) %>% 
  select(-td1, -td2) %>% 
  arrange(-brand_n_phn)

head(dfbrands)
```



brand_name   brand_url               brand_image_url                                    brand_n_phn
-----------  ----------------------  ------------------------------------------------  ------------
Samsung      samsung-phones-9.php    http://cdn2.gsmarena.com/vv/logos/lg_samsu.gif            1067
LG           lg-phones-20.php        http://cdn2.gsmarena.com/vv/logos/lg_lg.gif                560
Nokia        nokia-phones-1.php      http://cdn2.gsmarena.com/vv/logos/lg_nokia.gif             441
Motorola     motorola-phones-4.php   http://cdn2.gsmarena.com/vv/logos/lg_motor.gif             425
Alcatel      alcatel-phones-5.php    http://cdn2.gsmarena.com/vv/logos/lg_alcat2.gif            346
HTC          htc-phones-45.php       http://cdn2.gsmarena.com/vv/logos/lg_htc.gif               233

Extract the main color of each brand via the image.


```r
brand_color <- map_chr(dfbrands$brand_image_url, function(url){
  # url <- sample(dfbrands$brand_image_url, size = 1)
  # url <- "http://cdn2.gsmarena.com/vv/logos/lg_mmax.gif"
  img <- caTools::read.gif(url)
  
  colors <- count(data_frame(col = as.vector(img$image)), col) %>% 
    arrange(desc(n)) %>% 
    left_join(data_frame(hex = img$col, col = seq(length(img$col))),
              by = "col") %>% 
    filter(!is.na(hex) & !str_detect(hex, "#F[A-z0-9]F[A-z0-9]F[A-z0-9]"))
  
  str_sub(colors$hex[1], 0, 7)
  
})

dfbrands <- dfbrands %>% mutate(brand_color = brand_color)

n <- 30

dsbrands <- dfbrands %>% 
  head(n) %>% 
  mutate(x = brand_name,
         y = brand_n_phn) %>% 
  list.parse3()

tooltip <- tagList(
  tags$span(style = "float:right;color:#3C3C3C", "{point.y} models"),
  tags$br(),
  tags$img(src = '{point.brand_image_url}')
) %>% as.character()

highchart() %>% 
  hc_title(text = sprintf("Top %s Brands with more phone models", n)) %>% 
  hc_subtitle(text = "source: http://www.gsmarena.com/") %>% 
  hc_chart(zoomType = "x") %>% 
  hc_tooltip(
    useHTML = TRUE,
    backgroundColor = "white",
    borderWidth = 2,
    headerFormat = "<table style ='width:92px;height:22px' >",
    pointFormat = tooltip,
    footerFormat = "</table>"
  ) %>% 
  hc_xAxis(categories = map_chr(dsbrands, function(x) x$brand_name)) %>% 
  hc_add_series(data = dsbrands,
                showInLegend = FALSE,
                colorByPoint = TRUE,
                name = "phones models",
                type = "bar") %>% 
  hc_add_theme(
    hc_theme_merge(
      hc_theme_538(),
      hc_theme(colors = map_chr(dsbrands, function(x) x$brand_color))
      )
  )
```

<!--html_preserve--><div id="htmlwidget-9545" style="width:100%;height:500px;" class="highchart"></div>
<script type="application/json" data-for="htmlwidget-9545">{"x":{"hc_opts":{"title":{"text":"Top 30 Brands with more phone models"},"credits":{"enabled":false},"exporting":{"enabled":false},"plotOptions":{"series":{"turboThreshold":0}},"subtitle":{"text":"source: http://www.gsmarena.com/"},"chart":{"zoomType":"x"},"tooltip":{"useHTML":true,"backgroundColor":"white","borderWidth":2,"headerFormat":"<table style ='width:92px;height:22px' >","pointFormat":"<span style=\"float:right;color:#3C3C3C\">{point.y} models</span>\n<br/>\n<img src=\"{point.brand_image_url}\"/>","footerFormat":"</table>"},"xAxis":{"categories":["Samsung","LG","Nokia","Motorola","Alcatel","HTC","Micromax","Celkon","Philips","Huawei","Sony Ericsson","ZTE","BLU","Lenovo","Sagem","Spice","Asus","Sony","verykool","Allview","Siemens","Acer","BlackBerry","Plum","XOLO","Yezz","Panasonic","NEC","Pantech","Vodafone"]},"series":[{"data":[{"brand_name":"Samsung","brand_url":"samsung-phones-9.php","brand_image_url":"http://cdn2.gsmarena.com/vv/logos/lg_samsu.gif","brand_n_phn":1067,"brand_color":"#403684","x":"Samsung","y":1067},{"brand_name":"LG","brand_url":"lg-phones-20.php","brand_image_url":"http://cdn2.gsmarena.com/vv/logos/lg_lg.gif","brand_n_phn":560,"brand_color":"#8A8D8E","x":"LG","y":560},{"brand_name":"Nokia","brand_url":"nokia-phones-1.php","brand_image_url":"http://cdn2.gsmarena.com/vv/logos/lg_nokia.gif","brand_n_phn":441,"brand_color":"#0097D5","x":"Nokia","y":441},{"brand_name":"Motorola","brand_url":"motorola-phones-4.php","brand_image_url":"http://cdn2.gsmarena.com/vv/logos/lg_motor.gif","brand_n_phn":425,"brand_color":"#000000","x":"Motorola","y":425},{"brand_name":"Alcatel","brand_url":"alcatel-phones-5.php","brand_image_url":"http://cdn2.gsmarena.com/vv/logos/lg_alcat2.gif","brand_n_phn":346,"brand_color":"#000000","x":"Alcatel","y":346},{"brand_name":"HTC","brand_url":"htc-phones-45.php","brand_image_url":"http://cdn2.gsmarena.com/vv/logos/lg_htc.gif","brand_n_phn":233,"brand_color":"#97CB2F","x":"HTC","y":233},{"brand_name":"Micromax","brand_url":"micromax-phones-66.php","brand_image_url":"http://cdn2.gsmarena.com/vv/logos/lg_mmax.gif","brand_n_phn":230,"brand_color":"#F9C5A8","x":"Micromax","y":230},{"brand_name":"Celkon","brand_url":"celkon-phones-75.php","brand_image_url":"http://cdn2.gsmarena.com/vv/logos/lg_celkon.gif","brand_n_phn":229,"brand_color":"#FF0000","x":"Celkon","y":229},{"brand_name":"Philips","brand_url":"philips-phones-11.php","brand_image_url":"http://cdn2.gsmarena.com/vv/logos/lg_phili.gif","brand_n_phn":218,"brand_color":"#005AFF","x":"Philips","y":218},{"brand_name":"Huawei","brand_url":"huawei-phones-58.php","brand_image_url":"http://cdn2.gsmarena.com/vv/logos/lg_huawei.gif","brand_n_phn":195,"brand_color":"#E5B2B4","x":"Huawei","y":195},{"brand_name":"Sony Ericsson","brand_url":"sony_ericsson-phones-19.php","brand_image_url":"http://cdn2.gsmarena.com/vv/logos/lg_sonye.gif","brand_n_phn":188,"brand_color":"#437056","x":"Sony Ericsson","y":188},{"brand_name":"ZTE","brand_url":"zte-phones-62.php","brand_image_url":"http://cdn2.gsmarena.com/vv/logos/lg_zte.gif","brand_n_phn":183,"brand_color":"#235DAB","x":"ZTE","y":183},{"brand_name":"BLU","brand_url":"blu-phones-67.php","brand_image_url":"http://cdn2.gsmarena.com/vv/logos/lg_blu1.gif","brand_n_phn":167,"brand_color":"#7F7F7F","x":"BLU","y":167},{"brand_name":"Lenovo","brand_url":"lenovo-phones-73.php","brand_image_url":"http://cdn2.gsmarena.com/vv/logos/lg_lenovo.gif","brand_n_phn":138,"brand_color":"#003D7B","x":"Lenovo","y":138},{"brand_name":"Sagem","brand_url":"sagem-phones-13.php","brand_image_url":"http://cdn2.gsmarena.com/vv/logos/lg_sagem.gif","brand_n_phn":120,"brand_color":"#0000FF","x":"Sagem","y":120},{"brand_name":"Spice","brand_url":"spice-phones-68.php","brand_image_url":"http://cdn2.gsmarena.com/vv/logos/lg_spice.gif","brand_n_phn":120,"brand_color":"#B5AF9F","x":"Spice","y":120},{"brand_name":"Asus","brand_url":"asus-phones-46.php","brand_image_url":"http://cdn2.gsmarena.com/vv/logos/lg_asus.gif","brand_n_phn":117,"brand_color":"#2A4364","x":"Asus","y":117},{"brand_name":"Sony","brand_url":"sony-phones-7.php","brand_image_url":"http://cdn2.gsmarena.com/vv/logos/lg_sony.gif","brand_n_phn":111,"brand_color":"#000000","x":"Sony","y":111},{"brand_name":"verykool","brand_url":"verykool-phones-70.php","brand_image_url":"http://cdn2.gsmarena.com/vv/logos/lg_veryk.gif","brand_n_phn":104,"brand_color":"#F89F49","x":"verykool","y":104},{"brand_name":"Allview","brand_url":"allview-phones-88.php","brand_image_url":"http://cdn2.gsmarena.com/vv/logos/lg_allview2.gif","brand_n_phn":94,"brand_color":"#F7A0A4","x":"Allview","y":94},{"brand_name":"Siemens","brand_url":"siemens-phones-3.php","brand_image_url":"http://cdn2.gsmarena.com/vv/logos/lg_sieme.gif","brand_n_phn":94,"brand_color":"#00ABB5","x":"Siemens","y":94},{"brand_name":"Acer","brand_url":"acer-phones-59.php","brand_image_url":"http://cdn2.gsmarena.com/vv/logos/lg_acer.gif","brand_n_phn":93,"brand_color":"#83B817","x":"Acer","y":93},{"brand_name":"BlackBerry","brand_url":"blackberry-phones-36.php","brand_image_url":"http://cdn2.gsmarena.com/vv/logos/lg_bberry.gif","brand_n_phn":83,"brand_color":"#316C91","x":"BlackBerry","y":83},{"brand_name":"Plum","brand_url":"plum-phones-72.php","brand_image_url":"http://cdn2.gsmarena.com/vv/logos/lg_plum.gif","brand_n_phn":78,"brand_color":"#ECB4B2","x":"Plum","y":78},{"brand_name":"XOLO","brand_url":"xolo-phones-85.php","brand_image_url":"http://cdn2.gsmarena.com/vv/logos/lg_xolo1.gif","brand_n_phn":78,"brand_color":"#2F2B2C","x":"XOLO","y":78},{"brand_name":"Yezz","brand_url":"yezz-phones-78.php","brand_image_url":"http://cdn2.gsmarena.com/vv/logos/yezz-logo.gif","brand_n_phn":77,"brand_color":"#000000","x":"Yezz","y":77},{"brand_name":"Panasonic","brand_url":"panasonic-phones-6.php","brand_image_url":"http://cdn2.gsmarena.com/vv/logos/lg_panas.gif","brand_n_phn":75,"brand_color":"#000000","x":"Panasonic","y":75},{"brand_name":"NEC","brand_url":"nec-phones-12.php","brand_image_url":"http://cdn2.gsmarena.com/vv/logos/lg_nec.gif","brand_n_phn":73,"brand_color":"#000000","x":"NEC","y":73},{"brand_name":"Pantech","brand_url":"pantech-phones-32.php","brand_image_url":"http://cdn2.gsmarena.com/vv/logos/lg_pante.gif","brand_n_phn":72,"brand_color":"#D8EDEE","x":"Pantech","y":72},{"brand_name":"Vodafone","brand_url":"vodafone-phones-53.php","brand_image_url":"http://cdn2.gsmarena.com/vv/logos/lg_vodafone.gif","brand_n_phn":71,"brand_color":"#E52539","x":"Vodafone","y":71}],"showInLegend":false,"colorByPoint":true,"name":"phones models","type":"bar"}]},"theme":{"colors":["#403684","#8A8D8E","#0097D5","#000000","#000000","#97CB2F","#F9C5A8","#FF0000","#005AFF","#E5B2B4","#437056","#235DAB","#7F7F7F","#003D7B","#0000FF","#B5AF9F","#2A4364","#000000","#F89F49","#F7A0A4","#00ABB5","#83B817","#316C91","#ECB4B2","#2F2B2C","#000000","#000000","#000000","#D8EDEE","#E52539"],"chart":{"backgroundColor":"#F0F0F0","plotBorderColor":"#606063","style":{"fontFamily":"Roboto","color":"#3C3C3C"}},"title":{"align":"left","style":{"fontWeight":"bold"}},"subtitle":{"align":"left"},"xAxis":{"gridLineWidth":1,"gridLineColor":"#D7D7D8","labels":{"style":{"fontFamily":"Unica One, sans-serif","color":"#3C3C3C"}},"lineColor":"#D7D7D8","minorGridLineColor":"#505053","tickColor":"#D7D7D8","tickWidth":1,"title":{"style":{"color":"#A0A0A3"}}},"yAxis":{"gridLineColor":"#D7D7D8","labels":{"style":{"fontFamily":"Unica One, sans-serif","color":"#3C3C3C"}},"lineColor":"#D7D7D8","minorGridLineColor":"#505053","tickColor":"#D7D7D8","tickWidth":1,"title":{"style":{"color":"#A0A0A3"}}},"tooltip":{"backgroundColor":"rgba(0, 0, 0, 0.85)","style":{"color":"#F0F0F0"}},"legend":{"itemStyle":{"color":"#3C3C3C"},"itemHoverStyle":{"color":"#FFF"},"itemHiddenStyle":{"color":"#606063"}},"credits":{"style":{"color":"#666"}},"labels":{"style":{"color":"#D7D7D8"}},"legendBackgroundColor":"rgba(0, 0, 0, 0.5)","background2":"#505053","dataLabelsColor":"#B0B0B3","textColor":"#C0C0C0","contrastTextColor":"#F0F0F3","maskColor":"rgba(255,255,255,0.3)"},"conf_opts":{"global":{"Date":null,"VMLRadialGradientURL":"http =//code.highcharts.com/list(version)/gfx/vml-radial-gradient.png","canvasToolsURL":"http =//code.highcharts.com/list(version)/modules/canvas-tools.js","getTimezoneOffset":null,"timezoneOffset":0,"useUTC":true},"lang":{"contextButtonTitle":"Chart context menu","decimalPoint":".","downloadJPEG":"Download JPEG image","downloadPDF":"Download PDF document","downloadPNG":"Download PNG image","downloadSVG":"Download SVG vector image","drillUpText":"Back to {series.name}","invalidDate":null,"loading":"Loading...","months":["January","February","March","April","May","June","July","August","September","October","November","December"],"noData":"No data to display","numericSymbols":["k","M","G","T","P","E"],"printChart":"Print chart","resetZoom":"Reset zoom","resetZoomTitle":"Reset zoom level 1:1","shortMonths":["Jan","Feb","Mar","Apr","May","Jun","Jul","Aug","Sep","Oct","Nov","Dec"],"thousandsSep":" ","weekdays":["Sunday","Monday","Tuesday","Wednesday","Thursday","Friday","Saturday"]}},"type":"chart","fonts":["Roboto","Unica+One"],"debug":false},"evals":[]}</script><!--/html_preserve-->

## Data phones
asdas


```r
dfphones <- map_df(dfbrands$brand_url, function(burl){
  # burl <- "dell-phones-61.php" # burl <- "samsung-phones-9.php"
  extract_page_info <- function(pburl) {
    message(pburl)
    phns <- read_html(pburl) %>% 
      html_nodes(".makers > ul > li")
    data_frame(
      phn = html_node(phns, "a") %>% html_text(),
      phn_url = html_node(phns, "a") %>% html_attr("href"),
      phn_image_url = html_node(phns, "img") %>% html_attr("src")  
    )
  }
  
  # check if have pages
  pages <- file.path(url, burl) %>% 
    read_html() %>% 
    html_nodes(".nav-pages a")
  
  if (length(pages) > 0) {
    
    dres <- pages %>% 
      html_attr("href") %>% 
      file.path(url, .) %>% 
      map_df(extract_page_info)
    
  } else {
    
    dres <- extract_page_info(file.path(url, burl))
    
  }
  
  dres %>% 
    mutate(brand_url = burl)
  
})
```

```r
dfphonesinfo <- map_df(dfphones$phn_url, function(purl){
  # purl <- sample(dfphones$phn_url, size = 1);purl <- "samsung_galaxy_s5_mini-6252.php"
  message(purl)
  dfphn <- file.path(url, purl) %>% 
    read_html() %>% 
    html_table(fill = TRUE) %>% 
    map_df(function(t){
      c(t[1, 1]) %>% 
        cbind(rbind(as.matrix(t[1, 2:3]),
                    as.matrix(t[2:nrow(t), 1:2]))) %>% 
        as.data.frame(stringsAsFactors = FALSE) %>% 
        setNames(c("spec", "spec2", "value")) %>% 
        mutate(spec2 = str_replace_all(spec2, "Â", ""),
               key = paste(spec, spec2, sep = "_") %>% str_to_lower(),
               key = str_trim(key),
               key = str_replace(key, "_$", "_other"),
               key = str_replace_all(key, "\\.", "")) %>% 
        select(key, value)
    }) %>%
    distinct(key) %>% 
    spread(key, value) %>%
    mutate(phn_url = purl) 
})
```


```r
dfphns <- dfbrands %>% 
  right_join(dfphones, by = "brand_url") %>% 
  right_join(dfphonesinfo, by = "phn_url") 

dfphns <- dfphns %>% 
  mutate(body_dimensions = str_replace(body_dimensions, "mm \\(.*\\)", ""),
         weight = as.numeric(str_extract(body_weight, "\\d+"))) %>% 
  separate(body_dimensions, into = c("height", "width", "depth"), sep = " x ",
           remove = FALSE, convert = TRUE) %>% 
  mutate(height = as.numeric(height),
         width = as.numeric(width),
         depth = as.numeric(depth))


dfphns2 <- dfphns %>% 
  mutate(screen_body_ratio = str_extract(display_size, "\\d+\\.\\d+%"),
         screen_body_ratio = str_replace(screen_body_ratio, "%", ""),
         screen_body_ratio = as.numeric(screen_body_ratio),
         screen_ppi = str_extract(display_resolution, "~\\d+"),
         screen_ppi = as.numeric(str_replace(screen_ppi, "~", "")),
         talk_time = as.numeric(str_extract(`battery_talk time`, "\\d+")),
         camera = str_extract(camera_primary, ".* MP"),
         camera = as.numeric(str_replace(camera," MP", ""))) %>% 
  mutate(year = str_extract(launch_announced, "\\d{4}"),
         month = str_extract(launch_announced, paste(month.abb, collapse = "|")),
         month = ifelse(str_detect(launch_announced, "1Q|Q1"), "Jan", month),
         month = ifelse(str_detect(launch_announced, "2Q|Q2"), "Apr", month),
         month = ifelse(str_detect(launch_announced, "3Q|Q3"), "Jul", month),
         month = ifelse(str_detect(launch_announced, "4Q|Q4"), "Oct", month),
         month = ifelse(is.na(month), "Jan", month)) %>% 
  # Cancelled Not officially announced yet 
  filter(!(is.na(year) | is.na(month) | is.na(height))) %>%
  left_join(data_frame(month = month.abb, monthn = seq(12)), by = "month") %>% 
  mutate(launch_date = paste(year, monthn, 1, sep = "-"),
         launch_date = ymd(launch_date)) %>% 
  filter(screen_body_ratio < 100)

dfbrandcolors <- dfphns2 %>% 
  select(brand_name, brand_color) %>% 
  distinct() %>% 
  {setNames(.$brand_color, .$brand_name)}
```

```r
dfphns2 %>%
  select(launch_date, brand_name, height, 
         depth, screen_body_ratio, camera) %>% 
  gather(key, value, -launch_date, -brand_name) %>% 
  ggplot(aes(launch_date, value)) + 
  geom_point(aes(color = brand_name), alpha = 0.25) +
  geom_smooth(color = "black", size = 1.2, alpha = 0.5) + 
  scale_color_manual(values = dfbrandcolors) +
  facet_wrap(~key, scales = "free") + 
  ggtitle("Release date vs some characteristics") + 
  theme_fivethirtyeight() +
  theme(legend.position = "none")
```

![](readme_files/figure-html/unnamed-chunk-8-1.png)

```r
dsphns2 <- dfphns2 %>% 
  select(launch_date, height, brand_name, brand_color, 
         brand_image_url,
         phn, phn_image_url) %>% 
  mutate(x = datetime_to_timestamp(launch_date),
         y = height,
         color = brand_color) %>% 
  list.parse3() %>% 
  map(function(x){
    x$color <- paste("rgba(", paste0(t(col2rgb(x$color)), collapse = ", "), ",", 0.2, ")")
    x
  })

dsphns2iphones <- dsphns2[map_lgl(dsphns2, function(x) str_detect(x$phn, "iPhone") )]

dsphns2galaxy <- dsphns2[map_lgl(dsphns2, function(x) str_detect(x$phn, "Galaxy S\\d{1}") )]
  
fit <- loess(height ~ datetime_to_timestamp(launch_date),
             data = dfphns2) %>% 
  augment() %>% 
  tbl_df()

head(fit)
```



 height   datetime_to_timestamp.launch_date.    .fitted     .se.fit      .resid
-------  -----------------------------------  ---------  ----------  ----------
  121.4                         1.406851e+12   150.9912   0.9743322   -29.59116
  112.7                         1.398902e+12   149.7021   0.8500596   -37.00205
  112.7                         1.398902e+12   149.7021   0.8500596   -37.00205
  121.4                         1.404173e+12   150.5787   0.9268889   -29.17873
  210.0                         1.435709e+12   154.0727   1.8309032    55.92726
  129.7                         1.404173e+12   150.5787   0.9268889   -20.87873

```r
dssmooth <- fit %>% 
  select(x = datetime_to_timestamp.launch_date.,
         y = .fitted) %>% 
  distinct(x) %>% # imporant!
  arrange(x) %>% # really important!
  list.parse3()

dssarea <- fit %>% 
  mutate(x = datetime_to_timestamp.launch_date.,
         y = .fitted - .se.fit,
         z = .fitted + .se.fit) %>% 
  select(x, y, z) %>% 
  distinct(x) %>% # imporant!
  arrange(x) %>% # really important!
  list.parse2()

tooltip <- tagList(
  tags$span(style = "color:#3C3C3C", "{point.phn}"),
  tags$hr(),
  tags$img(src = '{point.brand_image_url}'),
  tags$br(),
  tags$img(src = '{point.phn_image_url}', width = "95%")
  ) %>% as.character()

highchart() %>% 
  hc_title(text = "Release date vs Heigth") %>% 
  hc_subtitle(text = "Woth the Local Polynomial Regression Fitting") %>% 
  hc_chart(zoomType = "xy") %>% 
  hc_plotOptions(series = list(turboThreshold = 6000)) %>% 
  hc_yAxis(labels = list(text = ""), min = 60, max = 170) %>%
  hc_xAxis(type = "datetime") %>% 
  hc_add_serie(data = dssmooth, name = "LOESS",
               type = "spline", lineWidth = 3, color = "#000",
               enableMouseTracking = FALSE) %>%
  hc_add_serie(data = dssarea, name = "LOESS (SE)",
               type = "arearange", fillOpacity = 0.25, color = "#c3c3c3",
               lineWidth = 1.5, enableMouseTracking = FALSE) %>% 
  hc_add_serie(data = dsphns2, type = "scatter",
               name = "All Phones",zIndex = -3) %>%
  hc_add_serie(data = dsphns2galaxy, type = "scatter",
               name = "Galaxy S",zIndex = -3) %>%
  hc_add_serie(data = dsphns2iphones, type = "scatter",
               name = "IPhones",zIndex = -3) %>%
  hc_tooltip(
    useHTML = TRUE,
    backgroundColor = "white",
    borderWidth = 4,
    headerFormat = "<table style ='width:160px;height:200px' >",
    pointFormat = tooltip,
    footerFormat = "</table>"
  ) %>% 
  hc_add_theme(hc_theme_538())
```

<!--html_preserve--><div id="htmlwidget-5687" style="width:100%;height:500px;" class="highchart"></div>

## Sessiong info


```r
sessionInfo()
```

```
## R version 3.2.2 (2015-08-14)
## Platform: i386-w64-mingw32/i386 (32-bit)
## Running under: Windows 7 (build 7601) Service Pack 1
## 
## locale:
## [1] LC_COLLATE=Spanish_Chile.1252  LC_CTYPE=Spanish_Chile.1252   
## [3] LC_MONETARY=Spanish_Chile.1252 LC_NUMERIC=C                  
## [5] LC_TIME=Spanish_Chile.1252    
## 
## attached base packages:
## [1] stats     graphics  grDevices utils     datasets  methods   base     
## 
## other attached packages:
##  [1] htmltools_0.3     broom_0.4.0       printr_0.0.4     
##  [4] ggthemes_3.0.1    ggplot2_2.0.0     lubridate_1.5.0  
##  [7] highcharter_0.2.0 tidyr_0.4.1       stringr_1.0.0    
## [10] magrittr_1.5      rvest_0.3.1       xml2_0.1.2       
## [13] purrr_0.2.1       dplyr_0.4.3      
## 
## loaded via a namespace (and not attached):
##  [1] DBI_0.3.1         bitops_1.0-6      lattice_0.20-33  
##  [4] formatR_1.2.1     htmlwidgets_0.5   parallel_3.2.2   
##  [7] viridisLite_0.1.1 labeling_0.3      Rcpp_0.12.3      
## [10] highr_0.5.1       plyr_1.8.3        httr_1.1.0       
## [13] tools_3.2.2       nlme_3.1-124      rmarkdown_0.9.5  
## [16] R6_2.1.2          zoo_1.7-12        selectr_0.2-3    
## [19] TTR_0.23-0        knitr_1.12.3      scales_0.3.0     
## [22] assertthat_0.1    curl_0.9.6        digest_0.6.9     
## [25] gtable_0.1.2      evaluate_0.8      Matrix_1.2-3     
## [28] stringi_1.0-1     reshape2_1.4.1    caTools_1.17.1   
## [31] munsell_0.4.3     grid_3.2.2        XML_3.98-1.3     
## [34] colorspace_1.2-6  data.table_1.9.6  psych_1.5.8      
## [37] xts_0.9-7         lazyeval_0.1.10   quantmod_0.4-5   
## [40] yaml_2.1.13       mgcv_1.8-11       rlist_0.4.5.1    
## [43] mnormt_1.5-3      jsonlite_0.9.19   chron_2.3-47
```


---
title: "readme.R"
author: "jkunst"
date: "Fri Feb 26 16:47:21 2016"
---