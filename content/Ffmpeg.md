📹ffmpeg

fast forward
mpeg

ففمپگ ساده دیلده
مولتیمدیانین آچار فرانسه‌سی دی 

فابریس بلارد طرفیندن
2000 ایلینده باشلانیپ

مولتیمدیانی ادیت، تبدیل، استریم
ریکورد، ری‌انکد، ترنسکد و... الیر،
هر فرمتده هر codec ده و هر container ده

video = codec + container

تقریبا اینترنتده هر پلتفرم
کی ویدیونان ایشی وار
ففمپگ ایشلدیر.
یوتیوب، نتفلیکس، پورن‌هاب، اینستاگرام و...

و تقریبا هر software کی
مولتیمدیانان ایشی وار
ففمپگ ایشلدیر
vlc, mpv
و ویدیو پلیرلرین/ادیتورلارین چوخو

🔄 تبدیل
ffmpeg   -i   input.mp4   output.mkv

آیری بی مثال
ffmpeg   -i   video.mp4   -vn   -c:a.  copy   audio.m4a

‼️processing is CPU-intensive
اونا گوره من c:v copy- چوخ ایشلدرم
چون استریمی کپی الیر و ری‌انکد المیر 


⚙️ بعضی پارامترلر

codec = coding-decoding

c-
codec

c:v ➡️ codec:video-
c:a ➡️ codec:audio-



vn ➡️ no video-
an ➡️ no audio-


ففمپگین بعضی آیری ابزارلاری دا واردی 
ffplay
ffprobe


بعضی عملی مثاللار

