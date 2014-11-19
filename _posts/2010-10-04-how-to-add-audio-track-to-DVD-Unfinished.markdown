---
layout: post
title:  "Как добавить новую звуковую дорожку на существующий DVD диск [не закончено]"
date:   2010-10-04 22:48:45
description: 
categories:
- blog
---

Мне, как любителю поизучать языки, иногда приходит в голову посмотреть тот или иной фильм с оригинальной звуковой дорожкой... А еще лучше и с субтитрами, поскольку на слух я не всегда хорошо разбираю... И лучше все это в DVD качестве... Но увы, найти готовый диск удовлетворяющий этим требованиям достаточно сложно... Особенно если речь идет не об английском языке... Можно найти DVD-диск с русским переводом, можно скачать что-то ужасно пожатое, с оригинальной дорожкой... Иногда удается найти отдельные субтитры, сотворенные неизвестным героем в .srt файле... Но вот как это собрать в конечный DVD... 

Собирать в конечный DVD мы будем средствами ОС GNU/Linux (мы же не идиоты почем зря платить деньги американским компаниям), при этом средствами консольными, потому как все визуальные инструменты в этой области не доросли до профессионального уровня, и используя их иной раз не представляется возможным понять а что именно сейчас делается...

Для работы нам понадобятся два пакета: DVDAuthor и ffmpeg. Первый из них умеет разбирать/собирать DVD-диск на составные части, а второй осуществлять манипуляции с полученными частями... Итак начнем...

## Разобрать

Перед началом работы надо "разобрать" исходный DVD на чистые mpeg'и, при помощи программы dvdunauthor входящий в пакет dvd author:

<pre>$ dvdunauthor /cdrom
DVDAuthor::dvdunauthor, version 0.6.14.
Build options: gnugetopt magick iconv freetype fribidi
Send bugs to <dvdauthor-users@lists.sourceforge.net>

libdvdread: Encrypted DVD support unavailable.
************************************************
**                                            **
**  No css library available. See             **
**  /usr/share/doc/libdvdread4/README.Debian  **
**  for more information.                     **
**                                            **
************************************************


INFO: VMGM


INFO: VTSM 1/7
STAT: [4] VOB 5, Cell 1 (97%)

INFO: VTS 1/7
STAT: [1] VOB 1, Cell 2 (95%)

[.... Ну тут еще некоторое количество похожих строк ...]

INFO: VTSM 7/7
STAT: [0] VOB 1, Cell 1 (0%)

INFO: VTS 7/7
STAT: [1] VOB 1, Cell 2 (100%, 0:00 remain)</pre>

dvdunauthor прелопатит весь диск в текущую директорию вычищая из .vob файлов всю информацию о структуре DVD. Информация о структуре (а возможно только ее часть, не знаю) будет записана в файл dvdauthor.xml, которым воспользуется программа dvdauthor когда мы будем собирать диск назад.

## Кто все эти люди?

Теперь не плохо бы разобраться, что за файлы на нас свалились: при помощи какого либо видео-плеера можно посмотреть на то какой фрагмент записан в каждом .vob файле:
<pre>$mplayer vob_02t_001.vob</pre>

А при помощи программы ffmpeg можно посмотреть на структуру каждого файла:
<pre> $ ffmpeg -i vob_02t_001.vob 
FFmpeg version SVN-r21686, Copyright (c) 2000-2010 Fabrice Bellard, et al.
  built on Feb 12 2010 09:47:34 with gcc 4.4.3
  configuration: [... опущено ...]

Seems stream 0 codec frame rate differs from container frame rate: 50.00 (50/1) -> 25.00 (25/1)
Input #0, mpeg, from 'vob_02t_001.vob':
  Duration: 01:25:56.03, start: 0.280000, bitrate: 5011 kb/s
    Stream #0.0[0x1e0]: Video: mpeg2video, yuv420p, 720x576 [PAR 16:15 DAR 4:3], 8500 kb/s, 25 fps, 25 tbr, 90k tbn, 50 tbc
    Stream #0.1[0x80]: Audio: ac3, 48000 Hz, stereo, s16, 192 kb/s
At least one output file must be specified</pre>

Из вывода видно что файл содержит два потока: первый видео, второй аудио. Если поискать по файлу dvdauthor.xml упоминание о vob_02t_001.vob то можно увидеть что оный единственный звуковой поток содержит в себе русскую звуковую дорожку:
<pre>&lt;audio lang="ru" format="ac3"/></pre>
Теперь наша задача добавить к существующим двум потокам, еще два: аудио поток с оригинальной французской озвучкой и поток субтитров

## Готовим звуковую дорожку (извлечение, синхронизация)

Теперь нам надо подготовить дорожку для добавления: во-первых ее надо отделить от того видео-файла вместе с котором она была добыта (озвучка к фильму редко когда добывается в виде отдельного файла), во-вторых синхронизировать ее со звуковой дорожкой нашего DVD-диска, и в третьих конвертировать ее в подобающий для DVD-диска формат.

Источником добавляемой звуковой дорожки у нас является семисот мегабайтный .avi-файл С безумно пожатым видео и аудио
<pre>$ ffmpeg -i film_in_french.avi 
FFmpeg version SVN-r21686, Copyright (c) 2000-2010 Fabrice Bellard, et al.
  built on Feb 12 2010 09:47:34 with gcc 4.4.3
  configuration: [... Опущено...]

Seems stream 0 codec frame rate differs from container frame rate: 30000.00 (30000/1) -> 25.00 (25/1)
Input #0, avi, from 'Le grand blond avec une chaussure noire [1972].avi':
  Duration: 01:24:59.12, start: 0.000000, bitrate: 1154 kb/s
    Stream #0.0: Video: mpeg4, yuv420p, 512x320, 25 fps, 25 tbr, 25 tbn, 30k tbc
    Stream #0.1: Audio: mp3, 24000 Hz, 2 channels, s16, 40 kb/s
At least one output file must be specified</pre>

Теперь легким движением руки извлекаем звуковую дорожку в обособленный звуковой файл

<b><pre>$ ffmpeg -i film_in_french.avi  -acodec copy french_track.mp3</pre></b><pre>
FFmpeg version SVN-r21686, Copyright (c) 2000-2010 Fabrice Bellard, et al.
  built on Feb 12 2010 09:47:34 with gcc 4.4.3
  configuration: [... Опушено ...]

Seems stream 0 codec frame rate differs from container frame rate: 30000.00 (30000/1) -> 25.00 (25/1)
Input #0, avi, from 'Le grand blond avec une chaussure noire [1972].avi':
  Duration: 01:24:59.12, start: 0.000000, bitrate: 1154 kb/s
    Stream #0.0: Video: mpeg4, yuv420p, 512x320, 25 fps, 25 tbr, 25 tbn, 30k tbc
    Stream #0.1: Audio: mp3, 24000 Hz, 2 channels, s16, 40 kb/s
Output #0, mp3, to 'french_track.mp3':
    Stream #0.0: Audio: libmp3lame, 24000 Hz, 2 channels, 40 kb/s
Stream mapping:
  Stream #0.1 -> #0.0
Press [q] to stop encoding
size=   24898kB time=5099.14 bitrate=  40.0kbits/s
video:0kB audio:24898kB global headers:0kB muxing overhead 0.000129%</pre>

Поскольку в нашем исходном видео файле музыка хранилась в достаточно общепринятом формате mp3, который поймет любой разумный звуковой-редактор, то мы можем позволить себе не перекодировать звук из формата в формат, а просто скоприровать его как есть (Опция -acodec copy как раз об этом и говорит: скопировать  звуковую дорожку в отдельный файл без преобразования формата). В результате получим звуковой файл french_track.mp3 (как заказали в последнем параметре командной строки)

Полученный файл содержит французскую озвучку фильма, однако накладывать его на существующий DVD-видео-файл как есть скорее всего нельзя. Судьба разных изданий фильмов на разных языках может быть разная, в результате подготовке к изданию из фильма может пропасть или может добавиться пара секунд видеоряда, что не повлияет на качество каждого издания, но в результате звуковые дорожки разных изданий оказываются рассинхронизированными...

Для синхронизации извлечем звуковую дорожку из нашего .vob-файла...

<b><pre>$ ffmpeg -i vob_02t_001.vob  russian_track.wav</pre></b><pre>
FFmpeg version SVN-r21686, Copyright (c) 2000-2010 Fabrice Bellard, et al.
  built on Feb 12 2010 09:47:34 with gcc 4.4.3
  configuration: [... Опущенно ...]

Seems stream 0 codec frame rate differs from container frame rate: 50.00 (50/1) -> 25.00 (25/1)
Input #0, mpeg, from 'vob_02t_001.vob':
  Duration: 01:25:56.03, start: 0.280000, bitrate: 5011 kb/s
    Stream #0.0[0x1e0]: Video: mpeg2video, yuv420p, 720x576 [PAR 16:15 DAR 4:3], 8500 kb/s, 25 fps, 25 tbr, 90k tbn, 50 tbc
    Stream #0.1[0x80]: Audio: ac3, 48000 Hz, stereo, s16, 192 kb/s
Output #0, wav, to 'russian_track.wav':
    Stream #0.0: Audio: pcm_s16le, 48000 Hz, stereo, s16, 1536 kb/s
Stream mapping:
  Stream #0.1 -> #0.0
Press [q] to stop encoding
size=  966762kB time=5156.06 bitrate=1536.0kbits/s
video:0kB audio:966762kB global headers:0kB muxing overhead 0.000004%</pre>

Как мы видем, ffmpeg прекрасно угадал тип выходного файла по расширению .wav (как установить тип выходного файла в .wav явно указав его в параметре -target я так и не разобрался, но это не важно)

Теперь вам следует открыть оба звуковых файла russian_track.wav и french_track.mp3 в вашем любимом звуковом редакторе (я рекомендую audacity) и убедиться что french_track и russian_track синхронизированны (то есть вся фоновая музыка и звуки, присутствующие в обоих файлах звучат одномременно), и при необходимости отредактировать franch_track для достижения требуемой синхронизации.

В случае, который я рассматриваю в качестве примера, мне понадобилось добавить нескротко секунд тишины в начале звуковой дорожки, и примерно пол секунды где-то в середине (ох как пришлось это место рассинхронизации поискать)

При сохранении модифицированной звуковой дорожки рекомендую сохранить его в формате .wav, дабы не преумножать потери от компресии. Для ясности пусть это будет файл french_track_mod.wav

## Добавляем звуковую дорожку

Для начала переконвертируем звуковую дорожку в подходящий для DVD-диска формат ac3. В качестве битрейта выберем 192k, как в русском саундтреке. Качетсва звучания это конечно не увеличит, исходный файл был сильно сжат mp3 кодеком, но зато и не ухудшит. А свободного места на DVD болванке хоть отбавляй

<b><pre>$ ffmpeg -i french_track_mod.wav -acodec ac3 -ab 192k french_track_mod.ac3</pre></b>
<pre>FFmpeg version SVN-r21686, Copyright (c) 2000-2010 Fabrice Bellard, et al.
  built on Feb 12 2010 09:47:34 with gcc 4.4.3
  configuration: [... Опущено ...]

Input #0, wav, from 'french_track_mod.wav':
  Duration: 01:25:00.22, bitrate: 1535 kb/s
    Stream #0.0: Audio: pcm_s16le, 48000 Hz, 2 channels, s16, 1536 kb/s
File 'french_track_mod.ac3' already exists. Overwrite ? [y/N] y
[ac3 @ 0x8437de0]No channel layout specified. The encoder will guess the layout, but it might be incorrect.
Output #0, ac3, to 'french_track_mod.ac3':
    Stream #0.0: Audio: ac3, 48000 Hz, stereo, s16, 192 kb/s
Stream mapping:
  Stream #0.0 -> #0.0
Press [q] to stop encoding
size=  119536kB time=5100.22 bitrate= 192.0kbits/s
video:0kB audio:119536kB global headers:0kB muxing overhead 0.000000%</pre>

Теперь дорожку в правильном формате добавляем в наш главный .vob файл

<b><pre>$ ffmpeg -i vob_02t_001.vob  -vcodec copy -acodec copy -i french_track_mod.ac3  -target pal-dvd vob_02t_001.new.vob -acodec copy -newaudio</pre></b>
<pre>FFmpeg version SVN-r21686, Copyright (c) 2000-2010 Fabrice Bellard, et al.
  built on Feb 12 2010 09:47:34 with gcc 4.4.3
  configuration: [... Опущено ...]

Seems stream 0 codec frame rate differs from container frame rate: 50.00 (50/1) -> 25.00 (25/1)
Input #0, mpeg, from 'vob_02t_001.vob':
  Duration: 01:25:56.03, start: 0.280000, bitrate: 5011 kb/s
    Stream #0.0[0x1e0]: Video: mpeg2video, yuv420p, 720x576 [PAR 16:15 DAR 4:3], 8500 kb/s, 25 fps, 25 tbr, 90k tbn, 50 tbc
    Stream #0.1[0x80]: Audio: ac3, 48000 Hz, stereo, s16, 192 kb/s
[ac3 @ 0x837ecc0]max_analyze_duration reached
[ac3 @ 0x837ecc0]Estimating duration from bitrate, this may be inaccurate
Input #1, ac3, from 'french_track_mod.ac3':
  Duration: 01:25:00.22, bitrate: 192 kb/s
    Stream #1.0: Audio: ac3, 48000 Hz, stereo, s16, 192 kb/s
Output #0, svcd, to 'vob_02t_001.new.vob':
    Stream #0.0: Video: mpeg2video, yuv420p, 720x576 [PAR 16:15 DAR 4:3], q=2-31, 8500 kb/s, 90k tbn, 25 tbc
    Stream #0.1: Audio: ac3, 48000 Hz, stereo, 192 kb/s
    Stream #0.2: Audio: ac3, 48000 Hz, stereo, 192 kb/s
Stream mapping:
  Stream #0.0 -> #0.0
  Stream #0.1 -> #0.1
  Stream #1.0 -> #0.2
Press [q] to stop encoding
frame=128902 fps=468 q=-1.0 Lsize= 3251998kB time=5100.22 bitrate=5223.4kbits/s
video:2971233kB audio:240382kB global headers:0kB muxing overhead 1.257423%</pre>

Нюансы расположения аргументов таковы: Входные файлы через -i записываются в начале, после них имя выходного файла, в самом конце &mdash; -newaudio, символизирущий тот факт что в выходном файле добавляется еще один звуковой трек, -acodec copy перед -newaudio говорит что новый трек будет добавлен без перекодирования; -vcodec copy -acodec copy после первого входного файла, так же говорит что в результирующий файл оба потока первого входного файлоа должны быть скопированны без перекодирования... -target pal-dvd указывает тип создаваемого файла. pal-dvd как раз подходит для  последующего применения dvdauthor для того чтобы преобразовать mpeg файл назад в dvd.


*To be continued...*

-- Main.DhyanNataraj - 2010-04-03
