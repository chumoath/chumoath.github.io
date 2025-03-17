# bitbake最小构建

## 1、单层

- BBPATH=.
- printhello.bb
- classes/base.bbclass
- conf/bitbake.conf

## 2、多层

- BBPATH=build
- meta-mylayer/printhello.bb
- meta-mylayer/classes/base.bbclass
- meta-mylayer/conf/bitbake.conf
- meta-mylayer/conf/layer.conf
- build/conf/bblayers.conf