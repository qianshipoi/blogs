---
title: Use NPOI Export excel
published: 2022-06-13
description: ''
image: ''
tags: ['Dotnet']
category: 'Develop'
draft: false
---

#### 原文地址：https://www.cnblogs.com/luckyting/articles/10563288.html

## 前端代码
     handlePopTrainClassStudents(index,row) {
          let link = document.createElement('a');
          link.style.display = 'none';
          link.href = 'http://localhost:15811/api/'+this.api['GetStudentInfoExcel']+'?tcId='+row.TC_ID;
          link.setAttribute('download','true');
          link.setAttribute('target','_blank')
          document.body.appendChild(link);
          link.click();
          document.body.removeChild(link);
        },
## 控制器


      public HttpResponseMessage GetStudentInfoExcel(int tcId)
            {
            FilesMake fm = new FilesMake();
            var stream = fm.ExportStudentByTcIdData(tcId);

            if (stream == null)
            {
                return new HttpResponseMessage(HttpStatusCode.NoContent);
            }

            try
            {
                HttpResponseMessage result = new HttpResponseMessage(HttpStatusCode.OK);
                result.Content = new StreamContent(stream);
                result.Content.Headers.ContentType = new MediaTypeHeaderValue("application/vnd.ms-excel");
                result.Content.Headers.ContentDisposition = new ContentDispositionHeaderValue("attachment");
                TrainClasses tc = db.TrainClasses.Find(tcId);
                result.Content.Headers.ContentDisposition.FileName = $"{tc.TC_Name}班学生详情-{System.DateTime.Now.ToString("yyyyMMddHHmmss")}.xls";
                return result;
            }
            catch
            {
                return new HttpResponseMessage(HttpStatusCode.NoContent);
            }
        }

## 处理代码

     public System.IO.Stream ExportStudentByTcIdData(int tcId)
        {
        #region 生成xls
        var lstTitle = new List<string> { "学生编号", "校区班级编号", "学号", "姓名", "姓名拼音",
            "性别", "身份证号", "学生状态", "已通过认证","学历","专业","毕业学校","个人手机","家庭坐机","QQ号",
            "通信地址","邮编","学历费","技能培养费","实训住宿费" ,"技术评价","班主任评价","备注","最终选择的岗位","选岗次数"};//,
        IWorkbook book = new HSSFWorkbook();
        ISheet sheet = book.CreateSheet("sheet1");
        IRow rowTitle = sheet.CreateRow(0);
        ICellStyle style = book.CreateCellStyle();
        style.VerticalAlignment = NPOI.SS.UserModel.VerticalAlignment.Center;//垂直居中
        for (int i = 0; i < lstTitle.Count; i++)
        {
            rowTitle.CreateCell(i).SetCellValue(lstTitle[i]);
        }

        var list = (from tcs in db.TrainClassStudents
                    join stu in db.Students on tcs.Student_ID equals stu.Student_ID into tcs_stu_tmp
                    from tcs_stu in tcs_stu_tmp.DefaultIfEmpty()
                    join tc in db.TrainClasses on tcs.TC_ID equals tc.TC_ID into tcs_stu_tc_tmp
                    from tcs_stu_tc in tcs_stu_tc_tmp.DefaultIfEmpty()
            where tcs_stu_tc.TC_ID == tcId
            select new
            {
                tcs_stu.Student_ID,
                tcs_stu.Student_Name,
                tcs_stu.SC_ID,
                tcs_stu.Student_NO,
                tcs_stu.Student_NameSpell,
                tcs_stu.Student_Sex,
                tcs_stu.Student_IdentityNumber,
                tcs_stu.Student_State,
                tcs_stu.Student_Exam,
                tcs_stu.Student_Education,
                tcs_stu.Student_Specialty,
                tcs_stu.Student_Schoolofgraduation,
                tcs_stu.Student_PersonalTel,
                tcs_stu.Student_FamilyTel,
                tcs_stu.Student_QQ,
                tcs_stu.Student_Address,
                tcs_stu.Student_PostCode,
                tcs_stu.Student_EducationMoney,
                tcs_stu.Student_SkillTrainingMoney,
                tcs_stu.Student_TrainResideMoney,
                tcs_stu.Student_Evaluate1,
                tcs_stu.Student_Evaluate2,
                tcs_stu.Student_Remark,
                tcs_stu.Station_ID,
                tcs_stu.Stationt_SelectStationCount
            }).ToList();
        if (list.Count > 0)
        {
            list.OrderByDescending(o => o.Student_ID);
            int start = 0;//记录同组开始行号
            int end = 0;//记录同组结束行号
            string temp = "";//记录上一行的值
            for (int i = 0; i < list.Count; i++)
            {
                IRow row = sheet.CreateRow(i + 1);
                row.CreateCell(0).SetCellValue(list[i].Student_ID);
                row.CreateCell(1).SetCellValue(list[i].SC_ID??0);
                row.CreateCell(2).SetCellValue(list[i].Student_NO);
                row.CreateCell(3).SetCellValue(list[i].Student_Name);
                row.CreateCell(4).SetCellValue(list[i].Student_NameSpell);
                row.CreateCell(5).SetCellValue(list[i].Student_Sex);
                row.CreateCell(6).SetCellValue(list[i].Student_IdentityNumber);
                row.CreateCell(7).SetCellValue(list[i].Student_State);
                row.CreateCell(8).SetCellValue(list[i].Student_Exam);
                row.CreateCell(9).SetCellValue(list[i].Student_Education);
                row.CreateCell(10).SetCellValue(list[i].Student_Specialty);
                row.CreateCell(11).SetCellValue(list[i].Student_Schoolofgraduation);
                row.CreateCell(12).SetCellValue(list[i].Student_PersonalTel);
                row.CreateCell(13).SetCellValue(list[i].Student_FamilyTel);
                row.CreateCell(14).SetCellValue(list[i].Student_QQ);
                row.CreateCell(15).SetCellValue(list[i].Student_Address);
                row.CreateCell(16).SetCellValue(list[i].Student_PostCode);
                row.CreateCell(17).SetCellValue(list[i].Student_EducationMoney);
                row.CreateCell(18).SetCellValue(list[i].Student_SkillTrainingMoney);
                row.CreateCell(19).SetCellValue(list[i].Student_TrainResideMoney);
                row.CreateCell(20).SetCellValue(list[i].Student_Evaluate1);
                row.CreateCell(21).SetCellValue(list[i].Student_Evaluate2);
                row.CreateCell(22).SetCellValue(list[i].Student_Remark);
                row.CreateCell(23).SetCellValue(list[i].Station_ID??0);
                row.CreateCell(24).SetCellValue(list[i].Stationt_SelectStationCount??0);

                row.GetCell(0, MissingCellPolicy.CREATE_NULL_AS_BLANK).SetCellType(CellType.String);
                var cellText = row.Cells[0].StringCellValue;//获取当前行 第1列的单元格的值

                if (cellText == temp)//上下行相等，记录要合并的最后一行
                {
                    end = i;
                }
                else//上下行不等，记录
                {
                    if (start != end)
                    {
                        //设置一个合并单元格区域，使用上下左右定义CellRangeAddress区域
                        //CellRangeAddress四个参数为：起始行，结束行，起始列，结束列
                        CellRangeAddress region = new CellRangeAddress(start + 1, end + 1, 0, 0);
                        sheet.AddMergedRegion(region);
                    }
                    start = i;
                    end = i;
                    temp = cellText;
                }
            }
        }

        #endregion
        for (int i = 0; i < lstTitle.Count; i++)
        {
            sheet.AutoSizeColumn(i);//i：根据标题的个数设置自动列宽
        }

        MemoryStream ms = new MemoryStream();
        book.Write(ms);
        ms.Seek(0, SeekOrigin.Begin);
        return ms;
    }
