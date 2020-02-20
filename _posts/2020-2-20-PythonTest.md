目录

1. 原理
2. 运行方法
3. 效果
4. 代码
1. 原理
正则匹配对相应字符串进行替换

2. 运行方法
python md_convert.py [a.md, b.md,...] # 转换给定文档
或
python md_convert.py # 转换目录下所有的md文档

3. 效果


4. 代码
#coding:utf-8

"""
----------------------------------------
description:
    md文档格式化处理类
    为md文档自动生成标题编号；
    如已有编号，更新之；
    支持 1-6 级标题；
    标题格式：##，###，####，#####，######

    如给定参数则转换给定文件；
    如未给定参数则转换目录下所有md文档。

author: sss

date:
----------------------------------------
change:
    
----------------------------------------

"""
__author__ = 'sss'

import os
import sys
import re



# 工具对象
def pr_type(i, info=''):
    print(info, i, type(i))


class MdDocProcessing(object):
    """
    md文档格式化处理类
    为md文档自动生成标题编号；
    如已有编号，自动更新；
    更新1-6 级标题；
    标题格式支持：##，###，####，#####，######

    如给定参数则转换给定文件；
    如未给定参数则转换目录下所有md文档。


    """
    def __init__(self):
        pass

    def start_processing(self, *ar):
        #self._command_check()

        doc_list = sys.argv[1:]

        if len(doc_list) == 0:
            doc_list = [x for x in os.listdir() if x.endswith('.md')]
            res = input("将转换以下文档：\n%s\nY(回车)/N?:" % doc_list)
            if res in 'yY':
                pass
            else:
                return
        #print(doc_list)

        for _ in doc_list:
            if not _.endswith(r'.md'):
                print('参数格式错误: %s '%(_), '\n请输入.md格式文件。')
                return

        for _ in doc_list:
            self.title_style_convert(_)
            print("文档<%s>转换完成。"%(_, ))

    def title_style_convert(self, file_name):
        """
        标题自动编号
        编号格式为： 1.1.1. 标题
        :param file_name:
        :return:
        """

        title_num_cur = [0]*6

        # 匹配正则表达式
        reg = r'^((#{2,6})[\s\d\.]{0,})(.*)$'
        pattern = re.compile(reg)

        # 输出文件对象
        res_file = open('_' + file_name, 'w', encoding='utf-8')

        with open(file_name, 'r', encoding='utf-8') as fi:
            line = fi.readline()
            while line:
                res = re.match(pattern, line)
                if res:
                    length = len(res.group(2))
                    title_num_cur, title_str = self._title_nuber_process(title_num_cur, length)
                    line = res.group(2) + ' ' + title_str + res.group(3) + '\n'

                res_file.write(line)
                line = fi.readline()
        # 关闭输出文件
        res_file.close()

        #os.rename(file_name, file_name + '.tmp')
        # 删除原文件
        os.remove(file_name)
        # 将输出文件改为原文件名
        os.rename('_' + file_name, file_name)
        return 

    def _command_check(self):
        #print(sys.argv)
        cmd_params = sys.argv[1:]
        if len(cmd_params) <= 0:
            self.help()
            exit()


    def _title_nuber_process(self, title_num, length=0):
        """
        根据目录级别及前一标题编号返回新编号
        :param title_num: 前一标题编号 list
        :param length: 标题级别 int
        :return: res: 标号 str
        """
        if length == 0:
            return ''

        title_num_z = [0]*6
        title_num[length-2] += 1
        title_num = title_num[:length-1] + title_num_z[length-1:]
        res = '.'.join(str(x) if x else '1' for x in title_num[:length-1]) + '. '
        return (title_num, res)

    @staticmethod
    def help():
        print('%s usage: need at least one param as markdown filename' % sys.argv[0])
        print('  python %s filename1 filename2 ... filenameN' % sys.argv[0])


def start():
    so = MdDocProcessing()
    so.start_processing()

def _test():
    so = MdDocProcessing()
    so.start_processing()
    pass



if __name__ == '__main__':
    #_test()
    start()

    pass
    # pr_type('s')
