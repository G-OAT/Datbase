# Datbase
ation.
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Text;
using System.Text.RegularExpressions;
@ -12,6 +12,9 @@ namespace StanSoft
    /// </summary>
    public class Article
    {
        /// <summary>
        /// 文章标题
        /// </summary>
        public string Title { get; set; }
        /// <summary>
        /// 正文文本
@ -21,6 +24,9 @@ namespace StanSoft
        /// 带标签正文
        /// </summary>
        public string ContentWithTags { get; set; }
        /// <summary>
        /// 文章发布时间
        /// </summary>
        public DateTime PublishDate { get; set; }
    }

@ -29,6 +35,7 @@ namespace StanSoft
    /// Date:   2012/12/30
    /// Update: 
    ///     2013/7/10   优化文章头部分析算法，优化
    ///     2014/4/25   添加Html代码中注释过滤的正则
    ///         
    /// </summary>
    public class Html2Article
@ -36,12 +43,14 @@ namespace StanSoft
        #region 参数设置

        // 正则表达式过滤：正则表达式，要替换成的文本
        private static readonly string[][] _filters = new string[][]{
                new string[] { @"(?is)<script.*?>.*?</script>", "" },
                new string[] { @"(?is)<style.*?>.*?</style>", "" },
                // 针对链接密集型的网站的处理，主要是门户类的网站，降低链接干扰
                new string[] { @"(?is)</a>", "</a>\n"}                 
            };
        private static readonly string[][] Filters =
        {
            new[] { @"(?is)<script.*?>.*?</script>", "" },
            new[] { @"(?is)<style.*?>.*?</style>", "" },
            new[] { @"(?is)<!--.*?-->", "" },    // 过滤Html代码中的注释
            // 针对链接密集型的网站的处理，主要是门户类的网站，降低链接干扰
            new[] { @"(?is)</a>", "</a>\n"}                 
        };

        private static bool _appendMode = false;
        /// <summary>
@ -105,7 +114,7 @@ namespace StanSoft
                body = m.ToString();
            }
            // 过滤样式，脚本等不相干标签
            foreach (var filter in Html2Article._filters)
            foreach (var filter in Filters)
            {
                body = Regex.Replace(body, filter[0], filter[1]);
            }
@ -125,7 +134,7 @@ namespace StanSoft
            Article article = new Article
            {
                Title = GetTitle(html),
                PublishDate = GetPublishDate(html),
                PublishDate = GetPublishDate(body),
                Content = content,
                ContentWithTags = contentWithTags
            };
@ -233,8 +242,10 @@ namespace StanSoft
                    }
                    result = Convert.ToDateTime(dateStr);
                }
                catch (Exception)
                { }
                catch (Exception ex)
                {
                    Console.WriteLine(ex);
                }
                if (result.Year < 1900)
                {
                    result = new DateTime(1900, 1, 1);
@ -268,6 +279,17 @@ namespace StanSoft
            StringBuilder sb = new StringBuilder();
            StringBuilder orgSb = new StringBuilder();

            StringBuilder sbDebug = new StringBuilder();
            for (int i = 0; i < lines.Length; i++)
            {
                sbDebug.Append(i);
                sbDebug.Append(',');
                sbDebug.Append(lines[i].Length);
                sbDebug.Append("\r\n");
            }
            File.WriteAllLines(Path.Combine(AppDomain.CurrentDomain.BaseDirectory, "data.txt"), lines, Encoding.Default);
            File.WriteAllText(Path.Combine(AppDomain.CurrentDomain.BaseDirectory, "stat.csv"), sbDebug.ToString());

            int preTextLen = 0;         // 记录上一次统计的字符数量
            int startPos = -1;          // 记录文章正文的起始位置
            for (int i = 0; i < lines.Length - _depth; i++)
