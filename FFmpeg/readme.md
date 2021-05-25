# Usando FFMPEG para várias coisas

# Vídeo
- concatenar videos sem recodificar:

```bash
#Dentro da pasta com os videos originais criamos um .txt com o nome de
#cada arquivo numa linha com o formato: file 'video.fmt' 
ffmpeg -f concat -i "lista_nomes.txt" -c copy "video_junto.mp4"
```
https://trac.ffmpeg.org/wiki/Concatenate


- compressões:

```bash
##ta faltando, vou adicionar aos poucos

#quanto maior o coeficiente crf maior a compressão, para compressão sem comprometimento visual perceptível indica-se 17~18 (crf 0 = lossless)
ffmpeg -i meu_video.mp4 -vcodec libx265 -crf 28 leve.mp4
ffmpeg -i out.mp4 -vcodec libx265 -crf 30 light.mp4
```


- gravação de tela:

```bash
#Descobrindo o tamanho da sua tela:
xdpyinfo | grep dimensions

#Exemplo 1
ffmpeg -video_size 1366x768 -framerate 25 -f x11grab -i :0.0+0,0 -f alsa -ac 2 -i hw:0 output.mkv
#		   tela				      L definindo pra gravar a tela inteira (da pra escolher uma area mexendo em "0,0" (x,y))

#Lossless (muito pesado)
ffmpeg -video_size 1366x768 -framerate 30 -f x11grab -i :0.0 -c:v libx264rgb -crf 0 -preset ultrafast output.mkv
#Para tentar reduzir podemos recodificar de forma mais eficaz, mas lenta (continuando lossless)
ffmpeg -i output.mkv -c:v libx264rgb -crf 0 -preset veryslow output-smaller.mkv
``` 

- cortar o ínicio/fim de um video sem recodificar:

```bash
ffmpeg -ss 00:00:40 -i output.mp4 -t 160 -c copy cut_output.mp4
```


- cortar em diversos pontos e recodificar um video:

```bash
ffmpeg -i uncut.mp4        -vf "select='between(t,115,150)+between(t,276,293)+between(t,350,418)',
		     setpts=N/FRAME_RATE/TB" out.mp4
```


- converter mp4 (ou outros) em mkv:

```bash
ffmpeg -i "input.mp4"	-vcodec copy -acodec copy	"output.mkv"
```

- converter vários arquivos do mesmo tipo de uma só vez:

```bash
for f in *.mkv; do ffmpeg -i "$f" -vcodec copy -acodec copy "${f%.*}.mp4"
```

- extrair legendas de um arquivo mkv:

```bash
ffmpeg -i "path.mkv" -map 0:s:0 name_sub.srt
				  0:s:1,2,3,...
```

- remover audio de um video sem recodificar

```bash	
#criando variáveis com nome dos arquivos
#(atenção, não tem espaço antes nem depois do "=")
input=example.mkv
output=example-nosound.mkv
#não esquecer o $
ffmpeg -i $input -c copy -an $output
```
ou, para remover tudo (legendas, etc.) menos o video:
```bash	   
ffmpeg -i $input -c:v copy -an $output			  
```


# Audio
- alterar a frequencia de um arquivo de audio:
```bash
# Primeiro, descobrimos a sample rate frequency dentre os metadados(xxx Hz):
ffmpeg -i "input_file"
ffmpeg -i "input_file.m4a" -af asetrate=44100*2,aresample=44100,atempo=0.25 "output.m4a"
#                                         L rate f em Hz    L rate f em Hz
```
