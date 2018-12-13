---
title: '.Net运用EPPlus导出Excel表'
id: UseEPPlusExportExcelTable
directory: Guide
tags:
  - ASP.NET
  - csharp
  - EPPlus
  - Excel
cover: https://up.enterdesk.com/edpic_source/10/b6/c1/10b6c1f5e1515e72f9347bf521780622.jpg
date: 2018-11-19 09:24:13
---
记录一次开发内容,前两天写了两个后台管理表，要求做一个导出成Excel的功能，这个是用后台使用C#写的，然后用了EPPlus框架导出了Excel表，用输出流返回就可以了，如下：
![](http://wx4.sinaimg.cn/large/0065B4vHgy1fxd8gn6v9xj31a40nzmyp.jpg)
![](http://wx3.sinaimg.cn/large/0065B4vHgy1fxd8grfev2j30y50ldgnp.jpg)
这样的两个订单功能及导出Excel，由于懒得把CSS和JS区分成文件，直接写在一个文件里面了，下面贴上代码，这是前端的：
```html
@using GL.Tools
@using Newtonsoft.Json
@using TengYe.Core.Data.Common.Enums
@model TengYe.Web.Portal.Areas.Business.Models.ResultTemplate
@{
    Layout = null;
    var mealOrderStats = ((IEnumerable<MealOrderState>) Enum.GetValues(typeof(MealOrderState))).Select(x => new
    {
        Text = x.GetDescription(),
        Value = (int)x
    });
}
<!DOCTYPE html>

<html>
<head>
    <title>外卖数据表子页面</title>
    <script src="https://cdn.bootcss.com/jquery/3.3.1/jquery.min.js"></script>
    <script src="https://cdn.bootcss.com/vue/2.5.17-beta.0/vue.min.js"></script>
    <script src="https://cdn.bootcss.com/axios/0.19.0-beta.1/axios.min.js"></script>
    <link href="~/Areas/Admin/Content/Scripts/layui/css/layui.css" rel="stylesheet" type="text/css"/>
    <style>
        * {
            border-width: 0px;
            margin: 0px;
            padding: 0px;
        }
        body {
            background-color: rgb(237,237,237)
        }
        .setting-area {
            width: 100%;
            height: 17vh;
            background-color: #009688;
            position: fixed;
            top: 0px;
            z-index: 1;
            text-align: center;
        }
        .setting-area .state-area {
            display: inline-block;
        }
        .setting-area .select-style {
            width: 200px;
            height: 38px;
        }
        .setting-area .datetime-area {
            display: inline-block;
        }
        .setting-area .datetime-area .datetime-wh {
            width: 200px;
        }
        .setting-area .search-area {
            height: 50px;
            line-height: 50px;
            display: inline-block;
        }
        .setting-area .page-area {
            text-align: center;
        }

        .item-area {
            width: 70%;
            margin: 17vh auto;
        }
        .item-area .card-wh {
            width: 100%;
            margin-top: 20px;
            border-radius: 13px;
            float: right;
        }
        .item-area .card-wh .head-style {
            border-radius: 13px 13px 0px 0px;
        }
        .item-area .card-wh .layui-card-header .order-basic {
            width: 100%;
        }
        .item-area .card-wh .layui-card-header .order-basic .basic-style {
            font-weight: bold;
            display: inline-block;
            width: 270px;
        }


        .text-left {
            text-align: left;
        }

        .text-right {
            text-align: right;
        }
        .item-area .card-wh .layui-card-body .item-content {
            margin-top: 10px;
        }
        .item-area .card-wh .layui-card-body .item-content .cover {
            width: 30px;
        }

        .item-area .card-wh .layui-card-body .item-content .basic-style {
            width: 250px;
            margin-left: 20px;
            display: inline-block;
        }
        .item-area .card-wh .layui-card-body .item-count {
            margin-top: 20px;
            text-align: center;
        }
        .item-area .card-wh .layui-card-body .item-count .basic-style {
            width: 320px;
            margin-left: 20px;
            margin-right: 20px;
            display: inline-block;
            color: rgb(231,166,47);
        }
        .item-area .card-wh .layui-card-body .item-count .state-style1 {
            color: #009688;
            font-size: 15px;
        }
    </style>
</head>
<body>
<div id="app">
    <!--工具区域-->
    <div class="setting-area" id="setting-container">
        <!--订单状态区域-->
        <div class="state-area layui-form">
            <div class="layui-input-inline">
                <select lay-filter="state">
                    <option value="">订单状态</option>
                    <option value="0">正在点餐</option>
                    <option value="1">未支付</option>
                    <option value="2">已支付</option>
                    <option value="3">商家已接单</option>
                    <option value="4">骑手已接单</option>
                    <option value="5">骑手取消接单</option>
                    <option value="6">配送中</option>
                    <option value="20">已完成</option>
                    <option value="21">已关闭</option>
                </select>
            </div>
        </div>
        <!--时间筛选区域-->
        <div class="datetime-area">
            <div class="layui-input-inline">
                <input class="layui-input datetime-wh" id="filter-date" placeholder="筛选订单日期时间" />
            </div>
        </div>
        <!--关键字搜索区域-->
        <div class="search-area">
            <div class="layui-input-inline">
                <input type="text" class="layui-input" v-model="keyWord" placeholder="订单号、联系人、手机" />
            </div>
        </div>
        <!--按钮区域-->
        <div>
            <button class="layui-btn layui-bg-orange" @@click="search">搜索</button>
            <button class="layui-btn layui-bg-orange" @@click="exportExcel">导出</button>
        </div>
        <!--页数区域-->
        <div class="page-area">
            <div id="page-container"></div>
        </div>
    </div>
    <!--item的容器-->
    <div class="item-area" id="item-container">
        <!--面板-->
        <div class="layui-card card-wh" v-for="order in orders">
            <!--面板头内容-->
            <div class="layui-card-header head-style">
                <div class="order-basic">
                    <div class="basic-style text-left">订单号：{{order.orderNo}}</div>
                    <div class="basic-style text-right">联系人：{{order.contact}}</div>
                    <div class="basic-style text-right">联系电话：{{order.phone}}</div>
                    <div class="basic-style text-right">成交时间：{{order.creationTime}}</div>
                </div>
            </div>
            <!--面板体内容-->
            <div class="layui-card-body">
                <div class="item-content" v-for="item in order.items">
                    <img :src="item.mealCover + '.70x60.jpg'" class="cover"/>
                    <div class="basic-style text-left">{{item.mealName}}</div>
                    <div class="basic-style text-left">单价金币：{{item.unitGoldCoin}}</div>
                    <div class="basic-style text-left">购买数量：{{item.number}}件</div>
                    <div class="basic-style text-right">小计：{{item.unitGoldCoin * item.number}}金币</div>
                </div>
                <!--面板统计内容-->
                <div class="item-count">
                    <hr />
                    <div class="basic-style"><b>总数量：{{order.numberCout}}件</b></div>
                    <div class="basic-style"><b>总计：{{order.unitGoldCoinCount}}金币</b></div>
                    <div class="state-style1"><b>订单状态：{{getStateName(order.state)}}</b></div>
                </div>
            </div>
        </div>
    </div>
</div>
<script src="~/Areas/Admin/Content/Scripts/layui/layui.js"></script>
<script>
    layui.use(['layer', 'form', 'laydate','laypage'], function() {
        var layer = layui.layer;
        var laydate = layui.laydate;
        var form = layui.form;
        var laypage = layui.laypage;

        var app = new Vue({
            el: "#app",
            data: {
                page: 1,
                pageSize: 10,
                first: false,
                orders: null,
                pages: null,
                keyWord: null,
                index: null,
                startDate: null,
                endDate: null,
                state: null,
                status: @Html.Raw(JsonConvert.SerializeObject(mealOrderStats)),
            },
            mounted: function() {
                var that = this;
                that.getOrders();
                form.render();
                form.on('select(state)', function(data){
                    that.state = data.value;
                });
            },
            methods: {
                getOrders: function() {
                    var that = this;
                    axios.get("@Url.Action("TabData", "TakeoutOrder")",
                            {
                                params: {
                                    page: that.page,
                                    pageSize: that.pageSize,
                                    keyWord: that.keyWord,
                                    startDate: that.startDate,
                                    endDate: that.endDate,
                                    state: that.state
                                }
                            })
                        .then(function(response) {
                            //请求成功的回调
                            that.orders = response.data.data;
                            that.pages = response.data.page;
                            that.index = response.data.index;
                            laypage.render({
                                elem: 'page-container',
                                limit: response.data.row,
                                count: response.data.dataCount,
                                curr: response.data.index,
                                jump: function (obj, first) {
                                    if (!first) {
                                        that.page = obj.curr;
                                        that.getOrders();
                                    }
                                }
                            });
                            layer.msg('刷新成功！');
                        })
                        .catch(function(error) {
                            layer.msg('请求错误！');
                        });
                },
                search: function() {
                    this.page = 1;
                    this.getOrders();
                },
                getStateName: function(state) {
                    var status = this.status;
                    for (var i = 0; i < status.length; i++) {
                        if (state == status[i].value) {
                            return status[i].text;
                        }
                    }
                },
                exportExcel: function() {
                    var that = this;
                    var url = "@Url.Action("ExportExcel", "TakeoutOrder")?keyWord=" +
                        escape(that.keyWord || "") +
                        "&startDate=" +
                        (that.startDate || "") +
                        "&endDate=" +
                        (that.endDate || "") +
                        "&state=" +
                        (that.state || "");
                    window.open(url);
                }
            }
        });
        //日期时间选择器
        laydate.render({
            elem: '#filter-date',//指定元素
            type: 'date',
            range: true,
            format: 'yyyy-MM-dd',
            zIndex: 99999999,
            done: function (value, date, endDate) {
                app.startDate = date.year + "-" + date.month + "-" + date.date;
                app.endDate = endDate.year + "-" + endDate.month + "-" + endDate.date;
            }
        });
    });
</script>
</body>
</html>
```
后台代码：
```csharp
using System;
using System.Data.Entity.Core.Common.CommandTrees;
using System.Data.Entity.Core.Metadata.Edm;
using System.Drawing;
using System.IO;
using System.Linq;
using System.Web.Mvc;
using GL.Tools;
using OfficeOpenXml;
using OfficeOpenXml.Style;
using TengYe.Core.Data;
using TengYe.Core.Data.Common.Enums;
using TengYe.Core.Data.Entities;
using TengYe.EF.Extends;
using TengYe.Web.Portal.Areas.Business.Models;

namespace TengYe.Web.Portal.Areas.Business.Controllers
{
    public class TakeoutOrderController : Controller
    {
        private readonly IDbContext _dbContext;

        public TakeoutOrderController(IDbContext dbContext)
        {
            _dbContext = dbContext;
        }

        public ActionResult Index()
        {
            return View();
        }

        // GET
        public JsonResult TabData(int page, int pageSize, string keyWord, DateTime? startDate, DateTime? endDate,
            MealOrderState? state)
        {
            endDate = endDate?.AddDays(1);
            var result = new ResultTemplate();
            result.Index = page;
            result.Row = pageSize;
            var datas = _dbContext.Set<MealOrder>()
                .Where(order => order.Type == MealOrderType.Takeout)
                .WhereIf(state != null, x => x.State == state)
                .WhereIf(startDate != null && endDate != null,
                    x => x.CreationTime >= startDate && x.CreationTime < endDate)
                .WhereIf(!string.IsNullOrEmpty(keyWord),
                    x => x.OrderNo == keyWord || x.Phone == keyWord || x.Contact == keyWord);
            var count = datas.Count();
            result.DataCount = count;
            result.Page = count == 0 ? 0 :
                count < pageSize ? 1 :
                count % result.Row == 0 ? count / result.Row : count / result.Row + 1;
            result.Data = datas.Select(order => new
                {
                    order.Id,
                    order.Address,
                    order.Contact,
                    order.OrderNo,
                    order.State,
                    order.Phone,
                    Items = order.Items.Select(x => new
                    {
                        x.Id,
                        x.MealName,
                        x.MealCover,
                        x.MemberId,
                        x.UnitGoldCoin,
                        x.Number,
                        x.MealOrderId
                    }),
                    NumberCout = order.Items.Sum(x => x.Number),
                    UnitGoldCoinCount = order.Items.Sum(x => x.UnitGoldCoin),
                    order.Type,
                    Rider = new
                    {
                        Id = order.Rider != null ? order.Rider.Id : (int?) null,
                        NickName = order.Rider != null ? order.Rider.NickName : string.Empty
                    },
                    PaymentMember = new
                    {
                        Id = order.PaymentMember != null ? order.PaymentMember.Id : (int?) null,
                        NickName = order.PaymentMember != null ? order.PaymentMember.NickName : string.Empty
                    },
                    Ids = order.ParticipatingMembers.Select(x => new
                    {
                        x.Id,
                        x.NickName,
                    }),
                    OrderState = order.State,
                    order.CreationTime
                }
            ).OrderByDescending(x => x.Id).Skip((page - 1) * pageSize).Take(pageSize).ToList();



            return new LowerCamelJsonResult(result);
        }

        public FileResult ExportExcel(string keyWord, DateTime? startDate, DateTime? endDate, MealOrderState? state)
        {
            endDate = endDate?.AddDays(1);
            var datas = _dbContext.Set<MealOrder>()
                .Where(order => order.Type == MealOrderType.Takeout)
                .WhereIf(state != null, x => x.State == state)
                .WhereIf(startDate != null && endDate != null,
                    x => x.CreationTime >= startDate && x.CreationTime < endDate)
                .WhereIf(!string.IsNullOrEmpty(keyWord),
                    x => x.OrderNo == keyWord || x.Phone == keyWord || x.Contact == keyWord);
            var result = datas.Select(order => new
                {
                    order.Id,
                    order.Address,
                    order.Contact,
                    order.OrderNo,
                    order.State,
                    order.Phone,
                    Items = order.Items.Select(x => new
                    {
                        x.Id,
                        x.MealName,
                        x.MealCover,
                        x.MemberId,
                        x.UnitGoldCoin,
                        x.Number,
                        x.MealOrderId
                    }),
                    NumberCout = order.Items.Sum(x => x.Number),
                    UnitGoldCoinCount = order.Items.Sum(x => x.UnitGoldCoin),
                    order.Type,
                    Rider = new
                    {
                        Id = order.Rider != null ? order.Rider.Id : (int?) null,
                        NickName = order.Rider != null ? order.Rider.NickName : string.Empty
                    },
                    PaymentMember = new
                    {
                        Id = order.PaymentMember != null ? order.PaymentMember.Id : (int?) null,
                        NickName = order.PaymentMember != null ? order.PaymentMember.NickName : string.Empty
                    },
                    Ids = order.ParticipatingMembers.Select(x => new
                    {
                        x.Id,
                        x.NickName,
                    }),
                    OrderState = order.State,
                    order.CreationTime
                }
            ).OrderByDescending(x => x.Id).ToList();

            using (MemoryStream strem = new MemoryStream()) 
            using (ExcelPackage package = new ExcelPackage(strem))
            {
                var worksheet = package.Workbook.Worksheets.Add("第一页");


                //Excel第一行列名
                worksheet.Cells[1, 1].Value = "订单号";
                worksheet.Cells[1, 2].Value = "联系人";
                worksheet.Cells[1, 3].Value = "联系电话";
                worksheet.Cells[1, 4].Value = "餐品";
                worksheet.Cells[1, 5].Value = "单价";
                worksheet.Cells[1, 6].Value = "数量";
                worksheet.Cells[1, 7].Value = "小计";
                worksheet.Cells[1, 8].Value = "总数量";
                worksheet.Cells[1, 9].Value = "总金额";
                worksheet.Cells[1, 10].Value = "订单时间";

                worksheet.Column(1).Width = 20;
                worksheet.Column(2).Width = 15;
                worksheet.Column(3).Width = 20;
                worksheet.Column(4).Width = 15;
                worksheet.Column(5).Width = 15;
                worksheet.Column(6).Width = 15;
                worksheet.Column(7).Width = 15;
                worksheet.Column(8).Width = 20;
                worksheet.Column(9).Width = 15;
                worksheet.Column(10).Width = 20;

                int row = 2;
                for (int i = 0; i < result.Count; i++)
                {
                    var item = result[i].Items.ToArray();
                    var lg = item.Length;

                    worksheet.Cells[row, 1, row + lg - 1, 1].Merge = true;
                    worksheet.Cells[row, 2, row + lg - 1, 2].Merge = true;
                    worksheet.Cells[row, 3, row + lg - 1, 3].Merge = true;
                    worksheet.Cells[row, 8, row + lg - 1, 8].Merge = true;
                    worksheet.Cells[row, 9, row + lg - 1, 9].Merge = true;
                    worksheet.Cells[row, 10, row + lg - 1, 10].Merge = true;

                    worksheet.Cells[row, 1].Value = result[i].OrderNo;
                    worksheet.Cells[row, 2].Value = result[i].Contact;
                    worksheet.Cells[row, 3].Value = result[i].Phone;
                    worksheet.Cells[row, 8].Value = result[i].NumberCout;
                    worksheet.Cells[row, 9].Value = result[i].UnitGoldCoinCount;
                    worksheet.Cells[row, 10].Value = result[i].CreationTime.ToString("yyyy-MM-dd HH:mm:ss");

                    for (int j = 0; j < item.Length; j++)
                    {
                        worksheet.Row(row).Height = 20;
                        worksheet.Cells[row, 4].Value = item[j].MealName;
                        worksheet.Cells[row, 5].Value = item[j].UnitGoldCoin;
                        worksheet.Cells[row, 6].Value = item[j].Number;
                        worksheet.Cells[row, 7].Value = item[j].Number * item[j].UnitGoldCoin;
                        row++;
                    }
                }

                worksheet.Cells[1, 1, 1, 10].Style.Font.Bold = true;
                worksheet.Cells[1, 1, 1, 10].Style.Font.Color.SetColor(Color.White);
                worksheet.Cells[1, 1, 1, 10].Style.Fill.PatternType = ExcelFillStyle.Solid;
                worksheet.Cells[1, 1, 1, 10].Style.Fill.BackgroundColor.SetColor(Color.FromArgb(0, 150, 136));
                worksheet.Cells[1, 1, row - 1, 10].Style.HorizontalAlignment = ExcelHorizontalAlignment.Center;//水平居中
                worksheet.Cells[1, 1, row - 1, 10].Style.VerticalAlignment = ExcelVerticalAlignment.Center;//垂直居中
                worksheet.Cells[1, 1, row - 1, 10].Style.Border.BorderAround(ExcelBorderStyle.Thin, Color.Black);

                package.Save();

                
                return File(strem.ToArray(), "application/vnd.openxmlformats-officedocument.spreadsheetml.sheet","外卖表.xlsx");
            }
        }
    }
}
```
另一张表，前端：
```html
@using GL.Tools
@using Newtonsoft.Json
@using TengYe.Core.Data.Common.Enums
@model TengYe.Web.Portal.Areas.Business.Models.ResultTemplate
@{
    Layout = null;
    var mealOrderStats = ((IEnumerable<MealOrderState>) Enum.GetValues(typeof(MealOrderState))).Select(x => new
    {
        Text = x.GetDescription(),
        Value = (int)x
    });
}
<!DOCTYPE html>

<html>
<head>
    <title>点餐数据表子页面</title>
    <script src="https://cdn.bootcss.com/jquery/3.3.1/jquery.min.js"></script>
    <script src="https://cdn.bootcss.com/vue/2.5.17-beta.0/vue.min.js"></script>
    <script src="https://cdn.bootcss.com/axios/0.19.0-beta.1/axios.min.js"></script>
    <link href="~/Areas/Admin/Content/Scripts/layui/css/layui.css" rel="stylesheet" type="text/css" />
    <style>
        * {
            border-width: 0px;
            margin: 0px;
            padding: 0px;
        }

        body {
            background-color: rgb(237,237,237)
        }

        .setting-area {
            width: 100%;
            height: 17vh;
            background-color: #009688;
            position: fixed;
            top: 0px;
            z-index: 1;
            text-align: center;
        }

        .setting-area .state-area {
            display: inline-block;
        }

        .setting-area .select-style {
            width: 200px;
            height: 38px;
        }

        .setting-area .datetime-area {
            display: inline-block;
        }

        .setting-area .datetime-area .datetime-wh {
            width: 200px;
        }

        .setting-area .search-area {
            height: 50px;
            line-height: 50px;
            display: inline-block;
        }

        .setting-area .page-area {
            text-align: center;
        }

        .item-area {
            width: 70%;
            margin: 17vh auto;
        }

        .item-area .card-wh {
            width: 100%;
            margin-top: 20px;
            border-radius: 13px;
            float: right;
        }

        .item-area .card-wh .head-style {
            border-radius: 13px 13px 0px 0px;
        }

        .item-area .card-wh .layui-card-header .order-basic {
            width: 100%;
        }

        .item-area .card-wh .layui-card-header .order-basic .basic-style {
            font-weight: bold;
            display: inline-block;
            width: 210px;
        }


        .text-left {
            text-align: left;
        }

        .text-right {
            text-align: right;
        }

        .text-center {
            text-align: center;
        }

        .item-area .card-wh .layui-card-body .item-content {
            margin-top: 10px;
        }

        .item-area .card-wh .layui-card-body .item-content .cover {
            width: 50px;
        }

        .item-area .card-wh .layui-card-body .item-content .basic-style {
            width: 235px;
            margin-left: 20px;
            display: inline-block;
        }

        .item-area .card-wh .layui-card-body .item-guys {
            margin-top: 10px;
        }
        .item-area .card-wh .layui-card-body .item-guys .guy {
            display: inline-block;
            width: 100px;
            text-align: center;
        }
        .item-area .card-wh .layui-card-body .item-guys .guy .cover {
            width: 40%;
            border-radius: 50px;
            border: 2px solid #009688;
        }
        .item-area .card-wh .layui-card-body .item-guys .guy .basic-style {
            width: 100%;
        }

        .item-area .card-wh .layui-card-body .item-count {
            margin-top: 20px;
            text-align: center;
        }

        .item-area .card-wh .layui-card-body .item-count .basic-style {
            width: 320px;
            margin-left: 20px;
            margin-right: 20px;
            display: inline-block;
            color: rgb(231,166,47);
        }

        .item-area .card-wh .layui-card-body .item-count .state-style1 {
            color: #009688;
            font-size: 15px;
        }
    </style>
</head>
<body>
    <div id="app">
        <!--工具区域-->
        <div class="setting-area" id="setting-container">
            <!--订单状态区域-->
            <div class="state-area layui-form">
                <div class="layui-input-inline">
                    <select lay-filter="state">
                        <option value="">订单状态</option>
                        <option value="0">正在点餐</option>
                        <option value="1">未支付</option>
                        <option value="2">已支付</option>
                        <option value="3">商家已接单</option>
                        <option value="4">骑手已接单</option>
                        <option value="5">骑手取消接单</option>
                        <option value="6">配送中</option>
                        <option value="20">已完成</option>
                        <option value="21">已关闭</option>
                    </select>
                </div>
            </div>
            <!--时间筛选区域-->
            <div class="datetime-area">
                <div class="layui-input-inline">
                    <input class="layui-input datetime-wh" id="filter-date" placeholder="筛选订单日期时间" />
                </div>
            </div>
            <!--关键字筛选区域-->
            <div class="search-area">
                <div class="layui-input-inline">
                    <input type="text" class="layui-input" v-model="keyWord" placeholder="订单号、联系人、手机" />
                </div>
            </div>
            <!--按钮区域-->
            <div>
                <button class="layui-btn layui-bg-orange" @@click="search">搜索</button>
                <button class="layui-btn layui-bg-orange" @@click="exportExcel">导出</button>
            </div>
            <!--页数区域-->
            <div class="page-area">
                <div id="page-container"></div>
            </div>
        </div>
        <!--item的容器-->
        <div class="item-area" id="item-container">
            <!--面板-->
            <div class="layui-card card-wh" v-for="order in orders">
                <!--面板头内容-->
                <div class="layui-card-header head-style">
                    <div class="order-basic">
                        <div class="basic-style text-left">订单号：{{order.orderNo}}</div>
                        <div class="basic-style text-center">餐桌：{{order.name}}</div>
                        <div class="basic-style text-center">支付用户：{{order.paymentMember.nickName==""?"暂无":order.paymentMember.nickName}}</div>
                        <div class="basic-style text-center">支付者ID：{{order.paymentMember.id==""?"暂无":order.paymentMember.id}}</div>
                        <div class="basic-style text-right">成交时间：{{order.creationTime}}</div>
                    </div>
                </div>
                <!--面板体内容-->
                <div class="layui-card-body">
                    <!--购买内容-->
                    <div class="item-content" v-for="item in order.items">
                        <img :src="item.mealCover + '.70x60.jpg'" class="cover" />
                        <div class="basic-style text-left">{{item.mealName}}</div>
                        <div class="basic-style text-left">单价金币：{{item.unitGoldCoin}}</div>
                        <div class="basic-style text-left">购买数量：{{item.number}}件</div>
                        <div class="basic-style text-right">小计：{{item.unitGoldCoin * item.number}}金币</div>
                    </div>
                    <!--参与购买的人-->
                    <hr v-if="order.ids.length"/>
                    <div class="item-guys">
                        <div class="guy" v-for="guy in order.ids">
                            <img :src="guy.avatar" class="cover" />
                            <div class="basic-style text-center">{{guy.nickName}}</div>
                        </div>
                    </div>
                    <!--面板统计内容-->
                    <div class="item-count">
                        <hr />
                        <div class="basic-style"><b>总数量：{{order.numberCout}}件</b></div>
                        <div class="basic-style"><b>总计：{{order.unitGoldCoinCount}}金币</b></div>
                        <div class="state-style1"><b>订单状态：{{getStateName(order.state)}}</b></div>
                    </div>
                </div>
            </div>
        </div>
    </div>
    <script src="~/Areas/Admin/Content/Scripts/layui/layui.js"></script>
    <script>
    layui.use(['layer', 'form', 'laydate','laypage'], function() {
        var layer = layui.layer;
        var laydate = layui.laydate;
        var form = layui.form;
        var laypage = layui.laypage;

        var app = new Vue({
            el: "#app",
            data: {
                page: 1,
                pageSize: 10,
                orders: null,
                pages: null,
                keyWord: null,
                index: null,
                startDate: null,
                endDate: null,
                state: null,
                status: @Html.Raw(JsonConvert.SerializeObject(mealOrderStats))
            },
            mounted: function () {
                var that = this;
                that.getOrders();
                form.render();
                form.on('select(state)', function (data) {
                    that.state = data.value;
                });
            },
            methods: {
                getOrders: function() {
                    var that = this;
                    axios.get("@Url.Action("TabData", "Order")",
                            {
                                params: {
                                    page: that.page,
                                    pageSize: that.pageSize,
                                    keyWord: that.keyWord,
                                    startDate: that.startDate,
                                    endDate: that.endDate,
                                    state: that.state
                                }
                            })
                        .then(function(response) {
                            //请求成功的回调
                            that.orders = response.data.data;
                            that.pages = response.data.page;
                            that.index = response.data.index;
                            laypage.render({
                                elem: 'page-container',
                                limit: response.data.row,
                                count: response.data.dataCount,
                                curr: response.data.index,
                                jump: function (obj, first) {
                                    if (!first) {
                                        that.page = obj.curr;
                                        that.getOrders();
                                    }
                                }
                            });
                            layer.msg('刷新成功！');
                        })
                        .catch(function(error) {
                            layer.msg('请求错误！');
                        });
                },
                search: function () {
                    this.page = 1;
                    this.getOrders();
                },
                getStateName: function (state) {
                    var status = this.status;
                    for (var i = 0; i < status.length; i++) {
                        if (state == status[i].value) {
                            return status[i].text;
                        }
                    }
                },
                exportExcel: function () {
                    var that = this;
                    var url = "@Url.Action("ExportExcel", "Order")?keyWord=" +
                        escape(that.keyWord || "") +
                        "&startDate=" +
                        (that.startDate || "") +
                        "&endDate=" +
                        (that.endDate || "") +
                        "&state=" +
                        (that.state || "");
                    window.open(url);
                }
            }
        });
        //日期时间选择器
        laydate.render({
            elem: '#filter-date',//指定元素
            type: 'date',
            range: true,
            format: 'yyyy-MM-dd',
            zIndex: 99999999,
            done: function (value, date, endDate) {
                app.startDate = date.year + "-" + date.month + "-" + date.date;
                app.endDate = endDate.year + "-" + endDate.month + "-" + endDate.date;
            }
        });
    });
    </script>
</body>
</html>
```
后端：
```csharp
using System;
using System.Drawing;
using System.IO;
using System.Linq;
using System.Web.Mvc;
using GL.Tools;
using OfficeOpenXml;
using OfficeOpenXml.Style;
using TengYe.Core.Data;
using TengYe.Core.Data.Common.Enums;
using TengYe.Core.Data.Entities;
using TengYe.EF.Extends;
using TengYe.Web.Portal.Areas.Business.Models;

namespace TengYe.Web.Portal.Areas.Business.Controllers
{
    public class OrderController : Controller
    {
        private readonly IDbContext _dbContext;

        public OrderController(IDbContext dbContext)
        {
            _dbContext = dbContext;
        }

        // GET
        public ActionResult Index()
        {
            return View();
        }

        public JsonResult TabData(int page, int pageSize, string keyWord, DateTime? startDate, DateTime? endDate, MealOrderState? state)
        {
            endDate = endDate?.AddDays(1);
            var result = new ResultTemplate();
            result.Index = page;
            result.Row = pageSize;
            var datas = _dbContext.Set<MealOrder>()
                .Where(order => order.Type == MealOrderType.Order)
                .WhereIf(state != null, x => x.State == state)
                .WhereIf(startDate != null && endDate != null,
                    x => x.CreationTime >= startDate && x.CreationTime < endDate)
                .WhereIf(!string.IsNullOrEmpty(keyWord),
                    x => x.OrderNo == keyWord || x.Phone == keyWord || x.Contact == keyWord);
            var count = datas.Count();
            result.DataCount = count;
            result.Page = count == 0 ? 0 : count < pageSize ? 1 : count % result.Row == 0 ? count / result.Row : count / result.Row + 1;
            result.Data = datas.Select(order => new
            {
                order.Id,
                order.Address,
                order.Contact,
                order.OrderNo,
                order.State,
                order.Phone,
                Items = order.Items.Select(x => new
                {
                    x.Id,
                    x.MealName,
                    x.MealCover,
                    x.MemberId,
                    x.UnitGoldCoin,
                    x.Number,
                    x.MealOrderId
                }),
                NumberCout = order.Items.Sum(x => x.Number),
                UnitGoldCoinCount = order.Items.Sum(x => x.UnitGoldCoin),
                order.Type,
                Rider = new
                {
                    Id = order.Rider != null ? order.Rider.Id : (int?)null,
                    NickName = order.Rider != null ? order.Rider.NickName : string.Empty
                },
                order.DiningTable.Name,
                PaymentMember = new
                {
                    Id = order.PaymentMember != null ? order.PaymentMember.Id : (int?)null,
                    NickName = order.PaymentMember != null ? order.PaymentMember.NickName : string.Empty,
                },
                Ids = order.ParticipatingMembers.Select(x => new
                {
                    x.Id,
                    x.NickName,
                    x.Avatar
                }),
                OrderState = order.State,
                /*SateName = order.State.GetDescription(),*/
                order.CreationTime
            }
            ).OrderByDescending(x => x.Id).Skip((page - 1) * pageSize).Take(pageSize).ToList();
            return new LowerCamelJsonResult(result);
        }

        public FileResult ExportExcel(string keyWord, DateTime? startDate, DateTime? endDate, MealOrderState? state)
        {
            endDate = endDate?.AddDays(1);
            var datas = _dbContext.Set<MealOrder>()
                .Where(order => order.Type == MealOrderType.Order)
                .WhereIf(state != null, x => x.State == state)
                .WhereIf(startDate != null && endDate != null, x => x.CreationTime >= startDate && x.CreationTime < endDate)
                .WhereIf(!string.IsNullOrEmpty(keyWord), x => x.OrderNo == keyWord || x.Phone == keyWord || x.Contact == keyWord)
                .ToList();
            
            var result = datas.Select(order => new
                {
                    order.Id,
                    order.Address,
                    order.Contact,
                    order.OrderNo,
                    order.State,
                    order.Phone,
                    Items = order.Items.Select(x => new
                    {
                        x.Id,
                        x.MealName,
                        x.MealCover,
                        x.MemberId,
                        x.UnitGoldCoin,
                        x.Number,
                        x.MealOrderId
                    }),
                    NumberCout = order.Items.Sum(x => x.Number),
                    UnitGoldCoinCount = order.Items.Sum(x => x.UnitGoldCoin),
                    order.Type,
                    Rider = new
                    {
                        Id = order.Rider != null ? order.Rider.Id : (int?) null,
                        NickName = order.Rider != null ? order.Rider.NickName : string.Empty
                    },
                    order.DiningTable.Name,
                    PaymentMember = new
                    {
                        Id = order.PaymentMember != null ? order.PaymentMember.Id : (int?) null,
                        NickName = order.PaymentMember != null ? order.PaymentMember.NickName : string.Empty,
                    },
                    Ids = order.ParticipatingMembers.Select(x => new
                    {
                        x.Id,
                        x.NickName,
                        x.Avatar
                    }),
                    OrderState = order.State,
                    //SateName = order.State.GetDescription(),
                    order.CreationTime
                }
            ).OrderByDescending(x => x.Id).ToList();

            using (MemoryStream strem = new MemoryStream())
            using (ExcelPackage package = new ExcelPackage(strem))
            {
                var worksheet = package.Workbook.Worksheets.Add("第一页");

                
                //Excel第一行列名
                worksheet.Cells[1, 1].Value = "订单号";
                worksheet.Cells[1, 2].Value = "餐桌号";
                worksheet.Cells[1, 3].Value = "餐品";
                worksheet.Cells[1, 4].Value = "单价金币";
                worksheet.Cells[1, 5].Value = "数量";
                worksheet.Cells[1, 6].Value = "小计";
                worksheet.Cells[1, 7].Value = "用户";
                worksheet.Cells[1, 8].Value = "总消费";
                worksheet.Cells[1, 9].Value = "订单时间";

                //Excel列宽
                worksheet.Column(1).Width = 20;
                worksheet.Column(2).Width = 15;
                worksheet.Column(3).Width = 20;
                worksheet.Column(4).Width = 15;
                worksheet.Column(5).Width = 15;
                worksheet.Column(6).Width = 15;
                worksheet.Column(7).Width = 15;
                worksheet.Column(8).Width = 15;
                worksheet.Column(9).Width = 20;
                
                int row = 2;
                //订单循环
                for (int i = 0; i < result.Count; i++)
                {
                    var items = result[i].Items.ToArray();
                    var ids = result[i].Ids.ToArray();

                    worksheet.Cells[row, 1, row + items.Length-1 , 1].Merge = true;
                    worksheet.Cells[row, 2, row + items.Length - 1, 2].Merge = true;
                    worksheet.Cells[row, 7, row + items.Length - 1, 7].Merge = true;
                    worksheet.Cells[row, 8, row + items.Length - 1, 8].Merge = true;
                    worksheet.Cells[row, 9, row + items.Length - 1, 9].Merge = true;

                    worksheet.Cells[row, 1].Value = result[i].OrderNo;
                    worksheet.Cells[row, 2].Value = result[i].Name;
                    worksheet.Cells[row, 8].Value = result[i].UnitGoldCoinCount;
                    worksheet.Cells[row, 9].Value = result[i].CreationTime.ToString("yyyy-MM-dd HH:mm:ss");


                    //订单用户循环
                    if (ids.Length > 0)
                    {
                        var guys = "";
                        foreach (var id in ids)
                        {
                            guys += id.NickName + ",";
                        }

                        guys = guys.Substring(0, guys.Length - 1);
                        worksheet.Cells[row, 7].Value = guys;
                    }
                    else
                    {
                        worksheet.Cells[row, 7].Value = "还未支付";
                    }

                    //订单详情循环
                    for (int j = 0; j < items.Length; j++)
                    {
                        worksheet.Row(row).Height = 20;
                        worksheet.Cells[row, 3].Value = items[j].MealName;
                        worksheet.Cells[row, 4].Value = items[j].UnitGoldCoin;
                        worksheet.Cells[row, 5].Value = items[j].Number;
                        worksheet.Cells[row, 6].Value = items[j].Number * items[j].UnitGoldCoin;
                        row++;
                    }
                }

                worksheet.Cells[1, 1, 1, 9].Style.Font.Bold = true;
                worksheet.Cells[1, 1, 1, 9].Style.Fill.PatternType = ExcelFillStyle.Solid;
                worksheet.Cells[1, 1, 1, 9].Style.Font.Color.SetColor(Color.White);
                worksheet.Cells[1, 1, 1, 9].Style.Fill.BackgroundColor.SetColor(Color.FromArgb(0,150,136));
                worksheet.Cells[1, 1, row-1, 9].Style.HorizontalAlignment = ExcelHorizontalAlignment.Center;//水平居中
                worksheet.Cells[1, 1, row-1, 9].Style.VerticalAlignment = ExcelVerticalAlignment.Center;//垂直居中
                worksheet.Cells[1,1,row-1,9].Style.Border.BorderAround(ExcelBorderStyle.Thin,Color.Black);

                package.Save();

                return File(strem.ToArray(), "application/vnd.openxmlformats-officedocument.spreadsheetml.sheet", "点餐表.xlsx");
            }
        }
    }
}
```
上面的注释已经详细的说明了EPPlus的使用方法，结合官方文档和查阅的资料，相信你很轻松就可以掌握。
<b><font color="FF0000">文章内容均为本人手写；<br>禁止转载，请尊重原著；<br>本人QQ：794763733。</font></b>