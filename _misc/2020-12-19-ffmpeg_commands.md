---
layout: cnblog_post
title: ffmpeg_commands
permalink: '/misc/ffmpeg_commands'
date: 2020-12-19 08:37:39
categories: misc
---



```
# 查看视频信息
ffprobe -select_streams v -show_entries format=duration,r_frame_rate,filename -show_streams \
-v quiet -of csv="p=0" -of json -i "/Users/mike/Downloads/XLDownload/5yoNmp5nnMQzl1E3.mp4"


```

ffmpeg-osx64-v4.2.2 -i a.mp4 -vn output.mp3

ffmpeg-osx64-v4.2.2 -i a.mp4 -vn output.mp3
ffmpeg -i a.mp4 -vn -acodec copy output.aac

# 添加图片水印
ffmpeg -i a.mp4 -i ../imgs/logo_10.png -filter_complex  "[1:v]scale=176:144[logo];[0:v][logo]overlay=x=10:y=10" out.mp4

# drawtext

ffmpeg -i a.mp4 -vf "drawtext=fontsize=100:fontfile='/System/Library/Fonts/STHeiti Light.ttc':text='汉字':x=20:y=20" out.mp4
# /System/Library/Fonts

ffmpeg -i a.mp4 -vf "drawtext=fontsize=100:fontfile='/System/Library/Fonts/STHeiti Light.ttc':text='汉字':x=20:y=20:fontcolor=green" out.mp4


ffmpeg -i a.mp4 -vf "drawtext=fontsize=100:fontfile='/System/Library/Fonts/STHeiti Light.ttc':text='汉字':x=20:y=20:fontcolor=green:box=1:boxcolor=yellow" out.mp4

ffmpeg -i a.mp4 -vf "drawtext=fontsize=100:fontfile='/System/Library/Fonts/STHeiti Light.ttc':text='汉字':x=20:y=20:fontcolor=green:box=1:boxcolor=yellow:enable=lt(mod(t\,3)\,1)" out.mp4

# gif

ffmpeg -y -i a.mp4 -ignore_loop 0 -i /Users/mike/Documents/zzResource/xxx.gif  -filter_complex overlay=0:H-h test_out2.mp4

# movie 滤镜

# 不能使用gif
ffmpeg -i a.mp4 -vf "movie=/Users/mike/Documents/zzResource/xxx.png[wm]; [in][wm]overlay=30:10[out]" out.mp4 


# 暂时不能用
ffmpeg -y -i a.mp4 -ignore_loop 0 -i /Users/mike/Documents/zzResource/xxx.gif  -filter_complex overlay=20:20 test_out2.mp4

# 
ffmpeg -i a.mp4 -ignore_loop 0 -i /Users/mike/Documents/zzResource/xxx.gif -filter_complex "[1:v]format=yuva444p,scale=80:80,setsar=1,rotate=PI/6:c=black@0:ow=rotw(PI/6):oh=roth(PI/6) [rotate];[0:v][rotate] overlay=(main_w-overlay_w)/2:(main_h-overlay_h)/2:shortest=1" -codec:a copy -y output.mp4

# 可以
ffmpeg -i a.mp4 -ignore_loop 0 -i /Users/mike/Documents/zzResource/xxx.gif -filter_complex "overlay=(main_w-overlay_w)/2:(main_h-overlay_h)/2:shortest=1" -y output.mp4


1. 下载视频a
2. a + gif水印 不同位置 -> b.mp4 (位置可变)
3. 用户加到尾印视频中 -> c.mp4 (位置可变)
4. b+c -> d.mp4 (需要用ffmpeg 处理淡入淡出效果)


ffmpeg -i a.mp4 -i /Users/mike/Documents/zzResource/xxx.gif -i /Users/mike/Documents/zzResource/xxx.gif -filter_complex \
"[0:v][1:v]overlay=25:(H-h)/2[bkg]; \
 [bkg][2:v]overlay=100:75" \
-c:a copy output.mp4


ffmpeg.exe -i input.mp4 -vf drawtext="fontsize=10:fontfile=/Windows/Fonts/arial.ttf:text='Text Here':x=if(eq(mod(n\,1200)\,0)\,rand(0\,(w-text_w))\,x):y=if(eq(mod(n\,1200)\,0)\,rand(0\,(h-text_h))\,y):enable=lt(mod(n\,1200)\,200)" -c:v libx264 -crf 17 -c:a copy output.mp4


ffmpeg -i a.mp4 -vf drawtext="fontsize=10:fontfile=/Windows/Fonts/arial.ttf:text='Text Here':x=if(eq(mod(n\,1200)\,0)\,rand(0\,(w-text_w))\,x):y=if(eq(mod(n\,1200)\,0)\,rand(0\,(h-text_h))\,y):enable=lt(mod(n\,1200)\,200)" -c:v libx264 -crf 17 -c:a copy output.mp4


ffmpeg -i a.mp4 -ignore_loop 0 -i /Users/mike/Documents/zzResource/xxx.gif -filter_complex "overlay=if(eq(mod(n\,300)\,0)\,rand(0\,(w-text_w))\,x):(main_h-overlay_h)/2:shortest=1" -y output.mp4

if(eq(mod(n\,300)\,0)\,rand(0\,(w-text_w))\,x)


# 闪烁 换位置 可以
ffmpeg -i a.mp4 -i /Users/mike/Documents/zzResource/xxx.gif -filter_complex "[1]trim=0:30,fade=in:st=0:d=1:alpha=1,fade=out:st=9:d=1:alpha=1,loop=999:750:0,setpts=N/25/TB[w];[0][w]overlay=shortest=1:x=if(eq(mod(n\,200)\,0)\,sin(random(1))*W\,x):y=if(eq(mod(n\,200)\,0)\,sin(random(1))*H\,y)" output.mp4


ffmpeg -i a.mp4 -ignore_loop 0 -i /Users/mike/Documents/zzResource/xxx.gif -filter_complex "overlay=shortest=1:x=if(eq(mod(n\,200)\,0)\,sin(random(1))*W\,x):y=if(eq(mod(n\,200)\,0)\,100\,y)" output.mp4 -y

ffmpeg -i a.mp4 -ignore_loop 0 -i /Users/mike/Documents/zzResource/xxx.gif -filter_complex "overlay=shortest=1:x=if(lt(n\,200)\,100\,main_w-overlay_w):y=if(lt(n\,200)\,100\,main_h-overlay_h)" output.mp4 -y


# gif + 文字水印
ffmpeg -i a.mp4 -ignore_loop 0 -i /Users/mike/Documents/zzResource/xxx.gif -filter_complex "[0:v][1:v]overlay=shortest=1:x=if(lt(n\,200)\,100\,main_w-overlay_w):y=if(lt(n\,200)\,100\,main_h-overlay_h)[bkg];[bkg]drawtext=fontsize=100:fontfile='/System/Library/Fonts/STHeiti Light.ttc':text='汉字':x=20:y=20:fontcolor=green" output.mp4 -y

ffmpeg -i a.mp4 -ignore_loop 0 -i /Users/mike/Documents/zzResource/xxx.gif -filter_complex \
"[0:v][1:v]overlay=shortest=1:x=if(lt(n\,200)\,100\,main_w-overlay_w):y=if(lt(n\,200)\,100\,main_h-overlay_h)[bkg];\
[bkg]drawtext=fontsize=100:fontfile='/System/Library/Fonts/STHeiti Light.ttc':text='汉字':x=20:y=20:fontcolor=green" output.mp4 -y

ffmpeg -i a.mp4 -ignore_loop 0 -i /Users/mike/Documents/zzResource/xxx.gif -filter_complex \
"[1:v]scale=120:120[scaled];\
[0:v][scaled]overlay=shortest=1:x=if(lt(n\,200)\,100\,main_w-overlay_w):y=if(lt(n\,200)\,100\,main_h-overlay_h)[logo];\
[logo]drawtext=fontsize=100:fontfile='/System/Library/Fonts/STHeiti Light.ttc':text='汉字':x=20:y=20:fontcolor=green" output.mp4 -y


ffmpeg -i a.mp4 -i ../imgs/logo_10.png -filter_complex  "[1:v]scale=176:144[scaled];[0:v][scaled]overlay=x=10:y=10[logo]" out.mp4


# 拼接视频

# 流程优化：
原来的流程
1. 将视频拆成 a.mp4 + concatenate-[ts].mp3
2. 生成尾部图片视频:
   username + nickname 与 hvideo_template.png 合成 尾部图片视频：pic-[ts].mp4
3. 给尾部图片视频添加淡入淡出效果，生成 effect-[ts].mp4
4. 源视频添加水印：
    logo 水印 + username 水印，前半段时间显示在左上角，后半段时间显示在右下角，生成 mark_video 对象
5. 将【步骤4】的结果 mark_video 和 【步骤3】的结果 effect-[ts].mp4 合成为无声的完整视频 concatenate-[ts].mp4
6. 将 【步骤5】结果 concatenate-[ts].mp4 与 【步骤1】生成的 concatenate-[ts].mp3 合成 result-[ts].mp4

现在流程
1. a + gif水印 不同位置 -> concatenate-[ts].mp4 (位置可变 ) (logo模式：gif模式，png模式)
2. 用户加到尾印视频中 -> effect-[ts].mp4 (位置可变) (尾印视频源： 图片模式 视频模式)
3. 【步骤1】+ 【步骤2】 -> result-[ts].mp4 (需要用ffmpeg 处理淡入淡出效果)
淡入淡出的效果 放在 步骤2 还是 步骤3？



2.

ffmpeg -loop 1 -i ../imgs/hvideo_template.png -c:v libx264 -t 1 -pix_fmt yuv420p -vf "[0:v]scale=1124:2000[scaled];[scaled]drawtext=fontsize=100:fontfile='/System/Library/Fonts/STHeiti Light.ttc':text='汉字':x=20:y=2" -r 30 out.mp4



ffmpeg -loop 1 -i ../imgs/hvideo_template.png -c:v libx264 -t 1 -pix_fmt yuv420p -vf "[0:v]scale=1124:2000[scaled];[scaled]drawtext=fontsize=100:fontfile='/System/Library/Fonts/STHeiti Light.ttc':text='汉字':x=20:y=20" -r 30 out.mp4


ffmpeg -loop 1 -i ../imgs/hvideo_template.png -i /Users/mike/Documents/zz3rdStone/projects/mars_media_process/commons/libs/watermark/imgs/portrait.jpg -c:v libx264 -t 1 -pix_fmt yuv420p -vf "[0:v]scale=1124:2000[scaled];[scaled][1:v]overlay=30:10[portrait];[portrait]drawtext=fontsize=50:fontfile='/System/Library/Fonts/STHeiti Light.ttc':fontcolor=#FFFFFF:text='nickname':x=(w-text_w)*0.5:y=305*3[nick];[nick]drawtext=fontsize=50:fontfile='/System/Library/Fonts/STHeiti Light.ttc':fontcolor=#FFFFFF:text='username-username':x=(w-text_w)*0.5:y=305*3+60" -r 30 out.mp4

-s

ffmpeg -loop 1 -i ../imgs/hvideo_template.png -i /Users/mike/Documents/zz3rdStone/projects/mars_media_process/commons/libs/watermark/imgs/portrait.jpg -c:v libx264 -t 1 -pix_fmt yuv420p -vf -s 1124x2000 "[0:v][1:v]overlay=30:10[portrait];[portrait]drawtext=fontsize=50:fontfile='/System/Library/Fonts/STHeiti Light.ttc':fontcolor=#FFFFFF:text='nickname':x=(w-text_w)*0.5:y=305*3[nick];[nick]drawtext=fontsize=50:fontfile='/System/Library/Fonts/STHeiti Light.ttc':fontcolor=#FFFFFF:text='username-username':x=(w-text_w)*0.5:y=305*3+60" -r 30 out.mp4


ffmpeg -loop 1 -i ../imgs/hvideo_template.png -i /Users/mike/Documents/zz3rdStone/projects/mars_media_process/commons/libs/watermark/imgs/portrait.jpg -c:v libx264 -t 1 -pix_fmt yuv420p -vf "[0:v]scale=1124:2000[scaled];[scaled][1:v]overlay=30:10[portrait];[portrait]drawtext=fontsize=50:fontfile='/System/Library/Fonts/STHeiti Light.ttc':fontcolor=#FFFFFF:text='nickname':x=(w-text_w)*0.5:y=305*3[nick];[nick]drawtext=fontsize=50:fontfile='/System/Library/Fonts/STHeiti Light.ttc':fontcolor=#FFFFFF:text='username-username':x=(w-text_w)*0.5:y=305*3+60" -r 30 out.mp4


#  添加 username nickname
ffmpeg -loop 1 -i ../imgs/hvideo_template.png -c:v libx264 -t 1 -pix_fmt yuv420p -vf "[0:v]scale=720:1280[scaled];[scaled]drawtext=fontsize=50:fontfile='/System/Library/Fonts/STHeiti Light.ttc':fontcolor=#FFFFFF:text='nickname':x=(w-text_w)*0.5:y=170*3[nick];[nick]drawtext=fontsize=50:fontfile='/System/Library/Fonts/STHeiti Light.ttc':fontcolor=#FFFFFF:text='username-username':x=(w-text_w)*0.5:y=170*3+60" -r 25 -c:a aac out.mp4 -y

# （同上面） 加声音
ffmpeg -loop 1 -i ../imgs/hvideo_template.png -f lavfi -i anullsrc -c:v libx264 -t 1 -pix_fmt yuv420p -vf "[0:v]scale=720:1280[scaled];[scaled]drawtext=fontsize=50:fontfile='/System/Library/Fonts/STHeiti Light.ttc':fontcolor=#FFFFFF:text='nickname':x=(w-text_w)*0.5:y=170*3[nick];[nick]drawtext=fontsize=50:fontfile='/System/Library/Fonts/STHeiti Light.ttc':fontcolor=#FFFFFF:text='username-username':x=(w-text_w)*0.5:y=170*3+60" -r 25 -c:a aac out.mp4 -y


# 添加头像
ffmpeg -i out.mp4 -i /Users/mike/Documents/zz3rdStone/projects/mars_media_process/commons/libs/watermark/imgs/portrait.jpg -filter_complex "[1:v]scale=90:90,geq=lum='p(X,Y)':a='st(1,pow(min(W/2,H/2),2))+st(3,pow(X-(W/2),2)+pow(Y-(H/2),2));if(lte(ld(3),ld(1)),255,0)'[portrait];[0:v][portrait]overlay=x=(main_w-w)*0.5:y=(main_h-h)*0.5" output-tail.mp4 -y


# username + nickname + 头像 （暂时不能用）
ffmpeg -loop 1 -i ../imgs/hvideo_template.png -i /Users/mike/Documents/zz3rdStone/projects/mars_media_process/commons/libs/watermark/imgs/portrait.jpg -f lavfi -i anullsrc -c:v libx264 -t 1 -pix_fmt yuv420p -vf "[0:v]scale=720:1040[scaled];[scaled]drawtext=fontsize=50:fontfile='/System/Library/Fonts/STHeiti Light.ttc':fontcolor=#FFFFFF:text='nickname':x=(w-text_w)*0.5:y=170*3[nick];[nick]drawtext=fontsize=50:fontfile='/System/Library/Fonts/STHeiti Light.ttc':fontcolor=#FFFFFF:text='username-username':x=(w-text_w)*0.5:y=170*3+60[nick_added];[1:v]scale=90:90,geq=lum='p(X,Y)':a='st(1,pow(min(W/2,H/2),2))+st(3,pow(X-(W/2),2)+pow(Y-(H/2),2));if(lte(ld(3),ld(1)),255,0)'[portrait];[nick_added][portrait]overlay=x=(main_w-w)*0.5:y=(main_h-h)*0.5" -r 25 -c:a aac out.mp4 -y

# 圆角测试
ffmpeg -i out.mp4 -i /Users/mike/Documents/zz3rdStone/projects/mars_media_process/commons/libs/watermark/imgs/portrait.jpg -filter_complex "[1:v]scale=90:90,geq=lum='p(X,Y)'[portrait];[0:v][portrait]overlay=x=(main_w-w)*0.5:y=(main_h-h)*0.5" output.mp4


# 拼接的三种方法
①ffmpeg -i "concat:output.mp4|output-tail.mp4" -c copy  result.mp4 -y

②不行
ffmpeg -y -f concat -i output.mp4 -i output-tail.mp4 -vcodec copy -an result.mp4

③
ffmpeg -f concat -safe 0 -i concact.txt -c copy result.mp4 -y

④加滤镜
ffmpeg -i output.mp4 -i output-tail.mp4 -filter_complex \
"[0:v][0:a][1:v][1:a] concat=n=2:v=1:a=1 [outv] [outa]" \
-map "[outv]" -map "[outa]" result.mp4 -y


ffmpeg -i output.mp4 -i output-tail.mp4 -filter_complex \
"[0][1] concat=n=2:v=1:a=1 [outv]" \
-map "[outv]" result.mp4 -y


ffmpeg -i output.mp4 -i output-tail.mp4 -filter_complex \
"[0:v][0:a][1:v][1:a] concat=n=2:v=1:a=1 [outv] [outa]" \
-map "[outv]" -map "[outa]" result.mp4 -y


# 淡入淡出效果
ffmpeg  -i output.mp4 -i output-tail.mp4 -filter_complex \
"[1:v]fade=t=in:st=0:d=1[v1]; \
[0:v][0:a][v1][1:a]concat=n=2:v=1:a=1[outv][outa]" \
-map "[outv]" -map "[outa]" final.mp4 -y


ffmpeg  -i a.mp4watermarked-1608555381136.mp4 -i pic-1608555381136.mp4 -f lavfi -t 0.1 -i anullsrc -filter_complex \
"[1:v]fade=t=in:st=0:d=1[v1]; \
[0:v][0:a][v1][2:a]concat=n=2:v=1:a=1[outv][outa]" \
-map "[outv]" -map "[outa]" final.mp4 -y


# 增加空音频
ffmpeg -i 1-Video.mp4 -i 2-openingtitle/EOTIntroFINAL640x480.mp4
       -i 3-videos/yelling.mp4 -i 4-endtitle/EOTOutroFINAL640x480.mp4
       -i 5-learnabout/Niambi640.mp4 -f lavfi -t 0.1 -i anullsrc -filter_complex
       "[0:v:0][5:a][1:v:0][1:a:0][2:v:0][2:a:0][3:v:0][3:a:0][4:v:0][5:a] concat=n=5:v=1:a=1 [v][a]"
       -map "[v]" -map "[a]" output_video.mp4


ffmpeg -i output.mp4 -i output-tail.mp4 -f lavfi -t 0.1 -i anullsrc -filter_complex \
"[0:v][0:a][1:v][2:a] concat=n=2:v=1:a=1 [outv] [outa]" \
-map "[outv]" -map "[outa]" result.mp4 -y


ffmpeg -i output.mp4 -i output-tail.mp4 -filter_complex \
"[0:v][1:v] concat=n=2:v=1:a=0 [outv]" \
-map "[outv]" result-nosound.mp4 -y



ffmpeg  -i a.mp4watermarked-1608568339552.mp4 -i pic-1608568339552.mp4 -f lavfi -i anullsrc -t 0.1 -s '720x1280' -filter_complex "[1:v]fade=t=in:st=0:d=1[v1]; [0:v][0:a][v1][2:a]concat=n=2:v=1:a=1[outv][outa]" -map "[outv]" -map "[outa]" final.mp4 -y

setdar=16/9

scale=1024:576:force_original_aspect_ratio=1
ffmpeg  -i a.mp4watermarked-1608555381136.mp4 -i pic-1608555381136.mp4 -f lavfi -i anullsrc -t 0.1 -filter_complex "[1:v]scale=720:1280:force_original_aspect_ratio=increase,crop=1280:720[scaled];[scaled]fade=t=in:st=0:d=1[v1]; [0:v][0:a][v1][2:a]concat=n=2:v=1:a=1[outv][outa]" -map "[outv]" -map "[outa]" final.mp4 -y

ffmpeg  -i a.mp4watermarked-1608555381136.mp4 -i pic-1608555381136.mp4 -f lavfi -i anullsrc -t 0.1 -filter_complex "[1:v]setdar=16/9[scaled];[scaled]fade=t=in:st=0:d=1[v1]; [0:v][0:a][v1][2:a]concat=n=2:v=1:a=1[outv][outa]" -map "[outv]" -map "[outa]" final.mp4 -y


ffmpeg  -i a.mp4watermarked-1608555381136.mp4 -i pic-1608555381136.mp4 -f lavfi -t 0.1 -i anullsrc -filter_complex "[1:v]fade=t=in:st=0:d=1[v1]; [0:v][0:a][v1][2:a]concat=n=2:v=1:a=1[outv][outa];[outv]scale=720:1280,setsar=sar=1[scaled]" -map "[scaled]" -map "[outa]" final.mp4 -y


ffmpeg  -i a.mp4watermarked-1608568339552.mp4 -i pic-1608568339552.mp4 -f lavfi -i anullsrc -t 0.1 -filter_complex "[1:v]scale=720:1280:force_original_aspect_ratio=increase,crop=720:1280,setsar=1[scaled];[scaled]fade=t=in:st=0:d=1[v1]; [0:v][0:a][v1][2:a]concat=n=2:v=1:a=1[outv][outa]" -map "[outv]" -map "[outa]" final.mp4 -y


scale=w=min(iw*720/ih\,1280):h=min(720\,ih*1280/iw), pad=w=640:h=480:x=(640-iw)/2:y=(480-ih)/2

ffmpeg -i pic-1608568339552.mp4 -filter:v scale=720:1280 pic-temp.mp4


## 
ffmpeg -i pic-1608568339552.mp4 -filter:v "scale=720:1280,setsar=1" pic-temp.mp4

ffmpeg  -i a.mp4watermarked-1608568339552.mp4 -i pic-temp.mp4 -f lavfi  -i anullsrc -filter_complex \
"[1:v]fade=t=in:st=0:d=1[v1]; \
[0:v][0:a][v1][2:a]concat=n=2:v=1:a=1[outv][outa]" \
-map "[outv]" -map "[outa]" final.mp4 -y



# 生成尾部图片视频
            # VideoTool().pic_to_video(video_image, pic_video, 25)
            # 尾部视频增加淡入淡出效果 生成effect_video
            # VideoTool().special_effects(pic_video, effect_video)

            # 给源视频添加水印，生成mark_video
            # mark_video, origin_video = NormalWatermark().video_add_text_new3(logo_path, video_path, user_name, "height", width, height)

            # 将mark_video 和 尾印视频拼接， 生成new_wm_video
            # 结果文件 concatenate-xx.mp4 (水印+尾印视频) 没有声音
            # VideoTool().concatenate_video(mark_video, effect_video, new_wm_video)
            # origin_video.close()


ffmpeg -loop 1 -i tailmark-1608603196946.bmp -f lavfi -i anullsrc -c:v libx264 -t 1 -pix_fmt yuv420p -vf "[0:v]scale=720:1280[scaled]" -r 25 -c:a aac tailmark.mp4 -y


ffmpeg  -i watermarked-1608603196946.mp4 -i tailmark.mp4 -filter_complex \
"[1:v]fade=t=in:st=0:d=1[v1]; \
[0:v][0:a][v1][1:a]concat=n=2:v=1:a=1[outv][outa]" \
-map "[outv]" -map "[outa]" final.mp4 -y


ffmpeg -loop 1 -i /Users/mike/Documents/zz3rdStone/projects/mars_media_process/commons/libs/watermark/videos/tailmark-1608605767432.bmp -f lavfi -i anullsrc         -c:v libx264 -t 3 -pix_fmt yuv420p -vf "[0:v]scale=720:1280[scaled]"         -r 25 -c:a aac /Users/mike/Documents/zz3rdStone/projects/mars_media_process/commons/libs/watermark/videos/tailmark-1608605767432.mp4 -y

ffmpeg -loop 1 -i /Users/mike/Documents/zz3rdStone/projects/mars_media_process/commons/libs/watermark/videos/tailmark-1608606165450.png -f lavfi -i anullsrc         -c:v libx264 -t 3 -pix_fmt yuv420p -vf "[0:v]scale=720:1280[scaled]"         -r 25 -c:a aac /Users/mike/
Documents/zz3rdStone/projects/mars_media_process/commons/libs/watermark/videos/tailmark-1608606165450.mp4 -y


b9901566b5b3876125094d170458093ec6d02f51
1a2c56100cbe82ab290f7e5d4da003d5d980c4c0

ffmpeg-osx64-v4.2.2 -i /Users/mike/Documents/zz3rdStone/projects/mars_media_process/commons/libs/watermark/videos/2.mp4 -i /Users/mike/Documents/zz3rdStone/projects/mars_media_process/commons/libs/watermark/imgs/logo_10.png -filter_cocd ../mplex "[1:v]scale=288:33[scaled];            [0:v][scaled]overlay=x=if(lt(n\,375)\,15\,main_w-overlay_w-15)            :y=if(lt(n\,375)\,15\,main_h-overlay_h-42)[logo];            [logo]drawtext=fontsize=22:fontfile='font/SF-Pro-Text-Semibold.otf':text='@Maymaymaytzy+':fontcolor=white:            x=if(lt(n\,375)\,15\,main_w-text_w-15):            y=if(lt(n\,375)\,53\,main_h-text_h-15)"             /Users/mike/Documents/zz3rdStone/projects/mars_media_process/commons/libs/watermark/videos/watermarked-1608622876829.mp4 -y



crop 1个
[[1322129, "oOxqypOjGXZvJVe8", 93345, "https://v.myclapper.com/57e0352avodtransusw1500000834/936c02b05285890810784378062/v.f169464.mp4"], {}, {"callbacks": null, "errbacks": null, "chain": null, "chord": null}]


logo 3个
[[1340979, "https://v.myclapper.com/4333d3f0voduse1500000834/8aa1ac1f5285890811098488967/f0.mp4"], {}, {"callbacks": null, "errbacks": null, "chain": null, "chord": null}]


[[1330530, "https://v.myclapper.com/4333d3f0voduse1500000834/d4adf6445285890811082618313/f0.mp4"], {}, {"callbacks": null, "errbacks": null, "chain": null, "chord": null}]

[[1340980, "https://v.myclapper.com/4333d3f0voduse1500000834/8aa1ac005285890811098488959/f0.mp4"], {}, {"callbacks": null, "errbacks": null, "chain": null, "chord": null}]

watermark
[[1322129, "oOxqypOjGXZvJVe8", 93345, "https://d1.myclapper.com/video_without_tiktok/oOxqypOjGXZvJVe8.mp4"], {}, {"callbacks": null, "errbacks": null, "chain": null, "chord": null}]


