---
title: ajax参数
date: 2019-09-21 00:00:00
tags: [ajax]
categories: JS及Jquery
---
$.ajax({
		url: &quot;com.fcb.erp.tools.util.runSql.biz.ext&quot;,
		type: &#39;POST&#39;,
		data: json,
		cache: false,
		//cache属性是true时：在第一次请求完成之后，如果地址和参数不变化，第二次去请求，
		//                                会默认获取缓存中的数据，不去读取服务器端的最新数据。
		//cache属性是false时：每次读取的是最新的数据。
		//ajax缓存只对GET方式的请求有效，因为浏览器认为POST请求提交的内容必定有变化，所以不走缓存。
		contentType:&#39;text/json&#39;,
		async: true/false,//异步还是同步 默认是true
		success: function (data) {
            console.info(data);
		}
});



## 联动
<div class="layui-form-item">
    <label class="layui-form-label">居住区域</label>
    <div class="layui-input-block">
        <select lay-verify="required" lay-filter="provSel" name="area">
            <option value=""></option>
            <c:forEach items="${areaList}" var="area">
            	<option id="area" value="${area.areaId}">${area.areaName}</option>
            </c:forEach>
        </select>
    </div>
    <label class="layui-form-label">居住楼宇</label>
    <div class="layui-input-block">
        <select id="build" name="build" lay-verify="required">
            <option value=""></option>
        </select>
    </div>
</div>


form.on('select(provSel)', function (data) {

    $.ajax({
        url: "<%=request.getContextPath()%>/AddrServlet",
        dataType: 'json',
        data: {
            action:"queryBuild",
            areaId: data.value
        },
        success: function (result) {
            var arr = eval(result);
	    $("#build option").remove();
            $.each(arr, function(key, val) {
                   $("#build").append("<option value='" + val.buildingId + "'>" + val.buildingName + "</option>")
            });
	    // 重新渲染，否则加载不出来
            form.render('select');
        }
    });
});




private void queryBuild(HttpServletRequest request, HttpServletResponse response) throws UnsupportedEncodingException {
    request.setCharacterEncoding("utf-8");
    response.setContentType("text/html;charset=UTF-8");

    String areaid = request.getParameter("areaId");
    String sql = "select * from building where area_id = ?";

    List<Building> buildingListlist = qr.query(JdbcUtil.getConnection(), sql, new BeanListHandler<>(Building.class), areaid);
    /*
    JSONArray jsonArray = new JSONArray();
    for(int i = 0;i < buildingListlist.size();i++){
        JSONObject jsonObject = new JSONObject();
        jsonObject.put("building_id",buildingListlist.get(i).getBuilding_id());
        jsonObject.put("building_name",buildingListlist.get(i).getBuilding_name());
        jsonArray.add(jsonObject);
    }
    */
    JSONArray jsonArray = JSONArray.toArray(buildingListlist);
    response.getWriter().print(jsonArray);
   
}
