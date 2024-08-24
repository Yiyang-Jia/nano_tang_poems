# nano_tang_poems
A tiny transformer model trained on Quan Tang Shi (全唐诗, a complete collection of poems of Tang dynasty),  in one Jupyter notebook.
Use m.generate_one_poem() to generate poems, where m is a model loaded from one of the .pt files.
Basic structure of the codes follow the nano GPT and tokenization YouTube tutorials of Andrei Karpathy. (A few differences in the Byte Pair Encoding codes to deal with Chinese characters).

There are 42889 poems (around 3.5 million Chinese characters) in the train + validation data set (not shown in this repo).
Data are tagged using special symbols '<' ,'>'  to delimit the beginning, ending of a poem,   and '|' to delimit the separation between a poem title and the corresponding poem body.  A snippet of two adjacent poems in the train data set looks like (by Li Shimin)

<出獵|楚王雲夢澤，漢帝長楊宮。豈若因農暇，閱武出轘嵩。三驅陳銳卒，七萃列材雄。寒野霜氛白，平原燒火紅。雕戈夏服箭，羽騎綠沉弓。怖獸潛幽壑，驚禽散翠空。長煙晦落景，灌木振嚴風。所為除民瘼，非是悅林叢。><冬狩|烈烈寒風起，慘慘飛雲浮。霜濃凝廣隰，冰厚結清流。金鞍移上苑，玉勒騁平疇。旌旗四望合，罝羅一面求。楚踣爭兕殪，秦亡角鹿愁。獸忙投密樹，鴻驚起礫洲。騎斂原塵靜，戈回嶺日收。心非洛汭逸，意在渭濱遊。禽荒非所樂，撫轡更招憂。> 

With vocabulary size around 8000 (fully randomized scenario gives cross entropy ~ 9.0),  at best I could achieve a validation loss of 4.53 (cross entropy). The smallest model that achieves this loss has 6.5M parameters, and  increasing model size no longer seems to reduce the optimal loss. 

# Example outputs
Most of the generated poems would not pass as one written by human as they break simple rules of poem writing, e.g.,

                古意                 (machine generated, fail)
                
        新蟬忽淒聲，月滿掌底鳴。
        
        去年今日光，巢起天邊生。
        
        今日舍農者，少光作兒兄。
        
        殘雞一食花，參枝獨宿鳴。

but occasionally it produces poems that can pass a (lenient) human grading (me). Here is one of the higher quality examples:

             贈韋郎中西山         (machine generated, pass)
        
        北風條葉落，滿眼天涯晴。
        
        水木泊何處，山鐘發故城。
        
        秋琴不倦臥，落筆忽忘形。
        
        不道千山夢，何人知酒平。

# Order of learning
I observed the following order of learning (among the relatively successful generated examples by human grading), i.e., what features of the training data are consecutively picked up as the model gets larger and loss gets smaller:

1) It learns the basic length pattern of poem lines, i.e.,  typically each line has two verses with equal number of characters.

2) It learns grammar (which is simple for poems), the distinction between verbs and nouns etc.  Actully this is picked up around the same time as 1),  I couldn't really tell which happened first.

3) It learns how to rhyme, namely it picks up phonetic information in the Chinese characters. 

4)  The meaning and imageries in the poem bodies start to make sense. This happens around the same time as 3), I couldn't really tell which happend first.

5)  The last and the most difficult thing to learn, is for the meaning of the title to match the meaning of the poem body. And it still does not do too well after the optimal loss is achieved.  However up to stage 2) there is absolutely no such matching observed.

6) One thing it never seems to have learned is a particular rule for rhyme:  one should not use two identical characters to rhyme,  and it is preferable not to use two different characters with the same pronunciations to rhyme (the latter is not a strict rule, see the snippet of the train data set I gave, where "宫" and “弓” are used to rhyme ).   This is reflected in the failed example I showed above (the character 鳴 appeared twice at the end of two verses).

# Causal masking contains positional information

Removing positional encoding layer still results in reasonable output,  thought significantly reducing efficiency.  Without any positional encoding layer, I can achieve optimal cross entropy loss of 4.65 with 8.1M parameters (compare with 4.53 with 6.5M parameters when positional encoding is present).  The following is one of the higher-quality example output, which still would not pass human grading:


     初山林宿，故鄉作      (machine generated without the positional encoding layer, fail)
     
     忽想雞林客，披襟煙景閑。
     
     悟談人罕玩，望客柳陰山。
     
     亂木長風起，餘花遠樹還。
     
     雲齊惹煙影，泉倒落花閑。
     
     歸路何時返，寒江若不關。

In this example, it is clear it learns all features of 1) to 5) discussed above,  just a little worse on each. 

If causal masking is also removed, I would get complete gibberish.
