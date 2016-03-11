****************************************
About.aspx
/******************************************


<%@ Page Title="About" Language="C#" MasterPageFile="~/Site.Master" AutoEventWireup="true" CodeBehind="About.aspx.cs" Inherits="DemoappRicha.About" %>

<asp:Content runat="server" ID="BodyContent" ContentPlaceHolderID="MainContent">
    
    <link rel="stylesheet" href="//code.jquery.com/ui/1.11.4/themes/smoothness/jquery-ui.css">
    <script src="//code.jquery.com/jquery-1.10.2.js"></script>
    <script src="//code.jquery.com/ui/1.11.4/jquery-ui.js"></script>
    

    <script>
        $(function () {
            $("#MainContent_btnAdd").bind("click", function () {
                var div = $("<div />");
                div.html(GetDynamicTextBox(""));
                $("#TextBoxContainer").append(div);
            });
            $("#MainContent_btnGet").bind("click", function () {
                var values = "";
                var firstDesc = $("#MainContent_txtDesc").val();
                values += firstDesc + "**";
                $("input[name=DynamicTextBox]").each(function () {
                    values += $(this).val() + "**";
                });
                var designation = $('#MainContent_txtautocom').val();
                SaveDetails(values, designation);
            });
            $("body").on("click", ".remove", function () {
                $(this).closest("div").remove();
            });
        });
        function GetDynamicTextBox(value) {
            return '<input name = "DynamicTextBox" type="text" value = "' + value + '" />&nbsp;' +
                    //'<input type="button" value="Remove" class="remove" />'
                '<asp:ImageButton class="remove" UseSubmitBehavior="false" OnClientClick="return false;"  runat="server" ImageUrl="~/Images/orderedList0.png" Height="20" Width="20" />'
        }
        function SaveDetails(values, designation)
        {
            $.ajax({
                url: '<%=ResolveUrl("~/About.aspx/SaveDetails") %>',
                data: "{ 'values': '" + values + "','designation':'"+ designation +"' }",
                dataType: "json",
                type: "POST",
                contentType: "application/json; charset=utf-8",
                success: function (data) {
                    alert('saved successfully.')
                },
                error: function (response) {
                    alert('error');
                },
                failure: function (response) {
                    alert('failure');
                }
            });
        }
        $(function () {
            $("#MainContent_txtautocom").autocomplete({
                source: function (request, response) {
                    $.ajax({
                        url: '<%=ResolveUrl("~/Default.aspx/GetCustomers") %>',
                        data: "{ 'prefix': '" + request.term + "'}",
                        dataType: "json",
                        type: "POST",
                        contentType: "application/json; charset=utf-8",
                        success: function (data) {
                            response($.map(data.d, function (item) {
                                return {
                                    label: item.split('-')[0],
                                    val: item.split('-')[1]
                                }
                            }))
                        },
                        error: function (response) {
                            alert(response.responseText);
                        },
                        failure: function (response) {
                            alert(response.responseText);
                        }
                    });
                },
                select: function (e, i) {
                    $("#MainContent_txtautocom").val(i.item.val);
                },
                minLength: 1
            });
        });






    </script>

    <aside>
        <h3>Aside Title</h3>
        <p>
            Use this area to provide additional information.
        </p>
        <ul>
            <li><a runat="server" href="~/">Home</a></li>
            <li><a runat="server" href="~/About.aspx">About</a></li>
            <li><a runat="server" href="~/Contact.aspx">Contact</a></li>
        </ul>
    </aside>
    <asp:Literal ID="litDesg" Text="Designation" runat="server" />

    <asp:TextBox ID="txtautocom" runat="server" />
    <asp:Button Text="Add Designation" ID="btnsave" onclick="btnsave_Click" runat="server" />
    <br />
    <asp:Literal ID="litDesc" Text="Job Description" runat="server" />
    <asp:TextBox ID="txtDesc" name="DynamicTextBox" runat="server" />

    <asp:ImageButton ID="btnAdd" OnClientClick="return false;"  UseSubmitBehavior="false" runat="server" ImageUrl="~/Images/orderedList1.png" Height="20" Width="20" />
    
    
    <asp:Button Text="Submit" ID="btnGet" UseSubmitBehavior="false" OnClientClick="return false;" runat="server"/>
<%--<input id="btnAdd" type="button" value="Add" />--%>
<br />
<br />
<div id="TextBoxContainer">
    <!--Textboxes will be added here -->
</div>
<br />
<%--<input id="btnGet" type="button" value="Get Values" />--%>



</asp:Content>


*************************************************
about.aspx END
-************************************************


*********************************
about.aspx.cs START
*********************************




using System;
using System.Collections.Generic;
using System.Configuration;
using System.Data.SqlClient;
using System.Linq;
using System.Web;
using System.Web.Services;
using System.Web.UI;
using System.Web.UI.WebControls;

namespace DemoappRicha
{
    public partial class About : Page
    {
        protected void Page_Load(object sender, EventArgs e)
        {

        }

        protected void btnsave_Click(object sender, EventArgs e)
        {
            var designationName = txtautocom.Text;
            using (SqlConnection conn = new SqlConnection())
            {
                conn.ConnectionString = ConfigurationManager.ConnectionStrings["itmall"].ConnectionString;
                using (SqlCommand cmd = new SqlCommand())
                {
                    cmd.CommandText = "IF NOT EXISTS (SELECT * FROM DesignationDetails where DesignationName=@SearchText) BEGIN INSERT INTO DesignationDetails values (@SearchText,NULL) END";
                    cmd.Parameters.AddWithValue("@SearchText", designationName);
                    cmd.Connection = conn;
                    conn.Open();
                    cmd.ExecuteNonQuery();
                    //using (SqlDataReader sdr = cmd.ExecuteReader())
                    //{
                    //    while (sdr.Read())
                    //    {
                    //        //customers.Add(string.Format("{0}-{1}", sdr["DesignationName"], sdr["Id"]));
                    //    }
                    //}
                    conn.Close();
                }
            }
        }

        [WebMethod]
        public static void SaveDetails(string values, string designation)
        {
            using (SqlConnection conn = new SqlConnection())
            {
                conn.ConnectionString = ConfigurationManager.ConnectionStrings["itmall"].ConnectionString;
                using (SqlCommand cmd = new SqlCommand())
                {
                    cmd.CommandText = "INSERT INTO DesignationDetails values (@Designation,@Description)";
                    cmd.Parameters.AddWithValue("@Description", values);
                    cmd.Parameters.AddWithValue("@Designation", designation);
                    cmd.Connection = conn;
                    conn.Open();
                    cmd.ExecuteNonQuery();
                    conn.Close();
                }
            }
        }

    }
}

******************************
about.aspx.cs END
*******************************

*****************************
Default.aspx.cs START
*****************************

using System;
using System.Collections.Generic;
using System.Configuration;
using System.Data.SqlClient;
using System.Linq;
using System.Web;
using System.Web.Services;
using System.Web.UI;
using System.Web.UI.WebControls;

namespace DemoappRicha
{
    public partial class _Default : Page
    {
        protected void Page_Load(object sender, EventArgs e)
        {

        }

        [WebMethod]
        public static string[] GetCustomers(string prefix)
        {
            List<string> customers = new List<string>();
            using (SqlConnection conn = new SqlConnection())
            {
                conn.ConnectionString = ConfigurationManager.ConnectionStrings["itmall"].ConnectionString;
                using (SqlCommand cmd = new SqlCommand())
                {
                    cmd.CommandText = "select Id, DesignationName from DesignationDetails where DesignationName like @SearchText + '%'";
                    cmd.Parameters.AddWithValue("@SearchText", prefix);
                    cmd.Connection = conn;
                    conn.Open();
                    using (SqlDataReader sdr = cmd.ExecuteReader())
                    {
                        while (sdr.Read())
                        {
                            customers.Add(string.Format("{0}-{1}", sdr["DesignationName"], sdr["Id"]));
                        }
                    }
                    conn.Close();
                }
            }
            return customers.ToArray();
        }
    }
}
***************************
default.aspx.cs END
***************************


****************************
Script START
************************

Script : 

IF NOT EXISTS (SELECT * FROM DesignationDetails where DesignationName='pp')
BEGIN
	INSERT INTO DesignationDetails values ('pp',NULL)
END

*****************************
SCRIPT END
***************************



