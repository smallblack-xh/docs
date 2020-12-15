店铺级别校验逻辑
1. 初始化校验组合串 filterString = ''
2. 根据``销售渠道大类``在字典``SHOPINFO_CHANNEL_ITEM``中查找对应的代号
   - `` ex: SHOPINFO_CHANNEL_ITEM=[{fieldName:ZK,fieldValue:0307},{fieldName:WX,fieldValue:01}]``
   - 判断是否属于``网销店``，网销店只需要``销售渠道第一级``判断即可，获得代号``WX``,此时``filterString = WX``
   - ``不属于网销店``销售渠道第二级为``折扣``时，获得代号``ZK``,此时``filterString = ZK``
   - ![20201110162826](https://cdn.jsdelivr.net/gh/smallblack-xh/images@main/blob/20201110162826.png)
3. 根据``店铺大类``在字典``SHOPINFO_SHOP_CATEGORY_ITEM``中查找对应的代号
   - ``ex: SHOPINFO_SHOP_CATEGORY_ITEM=[{fieldName:ZH,fieldValue:0504}]``
   - 那么店铺类别第二级为``折扣``时，获得代号``ZH``,``此时filterString = ZK-ZH``
   - ![20201110162926](https://cdn.jsdelivr.net/gh/smallblack-xh/images@main/blob/20201110162926.png)
4. ``销售渠道第二级``为``折扣``时，``非批发店需要结合销售渠道第三级``进一步判断，而``批发店需要结合店铺模式``进行判断
   - 非批发店 ：继续通过``SHOPINFO_CHANNEL_ITEM``查找代号，此时使用销售渠道小类查找
     ``ex: SHOPINFO_CHANNEL_ITEM=[{fieldName:LSTM,fieldValue:030703}]``
     那么销售渠道第三级为``临时特卖``时，获得代号``LSTM，此时filterString=ZK-ZH-LSTM``
     ![20201110163254](https://cdn.jsdelivr.net/gh/smallblack-xh/images@main/blob/20201110163254.png)
   - 批发店：通过``SHOPLEVEL_SHOPMODE_CONFIG``查找代号，此时使用店铺模式查找
     ``ex:SHOPLEVEL_SHOPMODE_CONFIG=[{fieldName:DM,fieldValue:U110101}]`` 那么店铺模式为``单品店``时，获得代号``DM``,``此时filterString=ZK-PF-DM``
     ![20201110163406](https://cdn.jsdelivr.net/gh/smallblack-xh/images@main/blob/20201110163406.png)
5. ``批发店销售渠道第二级为奥特莱斯``时，需要根据店铺模式进一步判断
     通过``SHOPLEVEL_SHOPMODE_CONFIG``查找代号，此时使用店铺模式查找
     ``ex:SHOPLEVEL_SHOPMODE_CONFIG=[{fieldName:DM,fieldValue:U110101}]`` 那么店铺模式为``单品店``时，获得代号``DM``,``此时filterString=AT-PF-DM``
     ![20201110163418](https://cdn.jsdelivr.net/gh/smallblack-xh/images@main/blob/20201110163418.png)
6. 通过``filterString``在字典``SHOPINFO_SHOPLEVEL_RELATIVE``查找对应的店铺级别的代号
     ``ex:SHOPINFO_SHOPLEVEL_RELATIVE=[{fieldName:ZK-ZH-LSTM,fieldValue:DPZK}]``
     filterString为``ZK-ZH-LSTM``时，获得代号``DPZK``
     filterString为``AT-PF-DM``时，获得代号``DPATA,DPATB``
     ![20201110163546](https://cdn.jsdelivr.net/gh/smallblack-xh/images@main/blob/20201110163546.png)
7. 使用6中获得的代号，对其进行``逗号分割``，再通过``SHOPINFO_SHOP_LEVEL``,找到对应的店铺级别编码
  ``ex:SHOPINFO_SHOP_LEVEL=[{fieldName:DPATA,fieldValue:U030109},{fieldName:DPATB,fieldValue:U030112}]``
  ``DPATA,DPATB -> [DPATA,DPATB] => [U030109,U030112]``
   ![20201110163558](https://cdn.jsdelivr.net/gh/smallblack-xh/images@main/blob/20201110163558.png)
8. 使用7中的结果对下拉数据进行置灰