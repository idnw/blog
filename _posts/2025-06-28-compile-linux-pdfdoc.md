---
layout: post
title: compile pdfdoc
subtitle: 编译linux内核文档
tags: [kernel, pdfdoc]
---

1.安装环境
dnf install ImageMagick dejavu-sans-fonts dejavu-serif-fonts graphviz google-noto-sans-cjk-ttc-fonts latexmk librsvg2-tools texlive-amscls texlive-amsfonts texlive-amsmath texlive-anyfontsize texlive-capt-of texlive-cmap texlive-collection-fontsrecommended texlive-collection-latex texlive-ec texlive-eqparbox texlive-euenc texlive-fancybox texlive-fancyvrb texlive-float texlive-fncychap texlive-framed texlive-luatex85 texlive-mdwtools texlive-multirow texlive-needspace texlive-oberdiek texlive-parskip texlive-polyglossia texlive-psnfss texlive-tabulary texlive-threeparttable texlive-titlesec texlive-tools texlive-ucs texlive-upquote texlive-wrapfig texlive-xecjk texlive-xetex-bin

dnf install -y python3-sphinx

python3 -m venv sphinx_latest

pip install -r ./Documentation/sphinx/requirements.txt

pip install --upgrade pip

dnf install texlive-ctex graphviz-gd

yum localinstall graphviz-gd-2.40.1-45.el8.x86_64.rpm

yum install https://rpmfind.net/linux/epel/9/Everything/x86_64/Packages/t/texlive-ctex-20200406-36.el9.noarch.rpm

2.编译命令
. sphinx_latest/bin/activate
make pdfdocs
make pdfdocs SPHINXDIRS="kernel-hacking scheduler"
deactivate
make cleandocs

3.输出目录
Documentation/output/*/pdf/*.pdf
