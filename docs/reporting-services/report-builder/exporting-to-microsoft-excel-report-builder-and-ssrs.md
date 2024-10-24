---
title: Export a paginated report to Microsoft Excel (Report Builder)
description: Find out about the Excel rendering extension, which you can use to export a Report Builder paginated report to the Office Open XML format for use in Microsoft Excel.
author: maggiesMSFT
ms.author: maggies
ms.date: 09/30/2024
ms.service: reporting-services
ms.subservice: report-builder
ms.topic: conceptual
ms.custom: updatefrequency5

# customer intent: As a SQL Server Reporting Services user, I want to see how the Excel rendering extension positions and organizes report data in Excel so that I can design my reports for successful exports.
---
# Export a paginated report to Microsoft Excel (Report Builder)

[!INCLUDE[Applies to](../../includes/ssrs-appliesto.md)] [!INCLUDE [Microsoft Report Builder (SSRS)](../../includes/ssrs-appliesto-ssrs-rb.md)] [!INCLUDE [Power BI Report Builder](../../includes/ssrs-appliesto-pbi-rb.md)] [!INCLUDE [Report Designer in SQL Server Data Tools](../../includes/ssrb-applies-to-ssdt-yes.md)]

When you work with Power BI or SQL Server Reporting Services (SSRS), you can use the Excel rendering extension to export paginated reports to [!INCLUDE[Microsoft Excel](../../includes/ofprexcel-md.md)]. The width of exported columns in Excel mimics the width of the columns in reports. But you can restructure the report data or further process it in a workbook.

This article discusses various aspects of the export process, such as renderer support for interactive features and the placement of data in cells. Limitations of Excel and the renderer are also covered.

## Export format

The Excel rendering extension exports reports into the Office Open XML format. The content type of files that the renderer generates is **application/vnd.openxmlformats-officedocument.spreadsheetml.sheet**, and the file extension is .xlsx.

You can change some default settings for this renderer by changing the device information settings. For more information, see [Excel device information settings](../../reporting-services/excel-device-information-settings.md).

For information about how to export a report in Excel format, see [Export paginated reports (Report Builder)](../../reporting-services/report-builder/export-reports-report-builder-and-ssrs.md).

> [!IMPORTANT]  
> When you define a report parameter of type `String`, the user is presented with a text box that can take any value. If the report parameter isn't tied to a query parameter and the parameter values are included in the report, there's a security risk. Specifically, the report user can enter expression syntax, script code, or a URL as the parameter value. As a result, the user can enter a malicious script or a malicious link. If the report is exported to Excel, other users can view it. If they select the rendered parameter contents, they risk inadvertently running a malicious script or going to a malicious site.
>
> To mitigate the risk of inadvertently running malicious scripts, open rendered reports only from trusted sources. For more information about securing reports, see [Secure reports and resources](../../reporting-services/security/secure-reports-and-resources.md).

## Excel limitations

Excel places limitations on exported reports due to the capabilities of Excel and its file formats. The following limitations are most significant:

- The maximum column width is 255 characters or 1726.5 points. The renderer doesn't verify that the column width is less than the limit.
- The maximum number of characters in a cell is 32,767. If this limit is exceeded, the renderer displays an error message.
- The maximum row height is 409 points. If the contents of a row cause the row height to exceed 409 points, the Excel cell shows a partial amount of text up to 409 points. The rest of the cell contents are still within the cell, up to the maximum allowed 32,767 characters.
- Because the maximum row height is 409 points, if the defined height of a cell in the report is greater than 409 points, Excel splits the cell contents into multiple rows.
- The maximum number of worksheets isn't defined in Excel. But external factors, such as memory and disk space, might cause limitations to be applied.
- In outlines, Excel permits up to seven nested levels only.
- The availability of an outline depends on the position of the report item that controls another item's visibility. An outline isn't available in the following cases:
  - The control report item isn't in the previous or next row relative to the item being expanded or collapsed.
  - The control report item isn't in the column of the item being expanded or collapsed.

For more information about Excel limitations, see [Excel specifications and limits](https://support.microsoft.com/office/excel-specifications-and-limits-1672b34d-7043-467e-8e27-269d656771c3).

### Text boxes and text

The following limitations apply to text boxes and text:

- Text box values that are expressions aren't converted to Excel formulas. The value of each text box is evaluated during report processing. The evaluated expression is exported as the contents of each Excel cell.
- Each text box is rendered within one Excel cell. For font size, font face, decoration, and font style, the formatting is supported on cell text.
- Excel doesn't support overline text formatting.
- Excel adds a default padding of approximately 3.75 points to the left and right sides of cells. If the padding of a text box is less than 3.75 points and the box isn't wide enough to accommodate the text, the text might wrap to a new line in Excel. To work around this issue, increase the width of the text box in the report.

### Images

The following limitations apply to images:

- Background images for report items are ignored because Excel doesn't support background images for individual cells.
- The Excel rendering extension only supports the background image of the report body. If a report body background image is displayed in the report, the image is rendered as a worksheet background image.

### Rectangles

The following limitation applies to rectangles: Rectangles in report footers aren't exported to Excel. But rectangles in the report body, tablix cells, and other similar components are rendered as a range of Excel cells.

### Report headers and footers

The following limitations apply to report headers and footers:

- Excel headers and footers support a maximum of 256 characters including markup. The rendering extension truncates the string at 256 characters.
- SSRS doesn't support margins on report headers and footers. In Excel, these margin values are set to zero.
- When you print a report that's exported to Excel, your printer settings can affect the rendering. Specifically, if a header or footer in the report contains multiple rows of data, you might not see multiple rows in the printout.
- Text boxes in a header or footer maintain their formatting but not their alignment when they're exported to Excel. Leading and trailing spaces are trimmed when the report is rendered to Excel, which changes the alignment.

### Merged cells

The following limitation applies to merging cells: If cells are merged, text isn't wrapped correctly.

The Excel renderer is primarily a layout renderer. Its goal is to replicate the layout of the rendered report as closely as possibly in an Excel worksheet. As a result, cells might be merged in the worksheet to preserve the report layout. Merged cells can cause problems because the sort functionality in Excel requires cells to be merged in a specific way for sorting to work properly. For example, if you want to sort a range of cells, Excel requires each merged cell in the range to have the same size as the other merged cells in the range.

Reducing the number of merged cells in your Excel worksheets makes it easier to sort worksheets. The following points can help you minimize the number of cells that get merged during the export process.

- The most common reason that cells get merged is that items aren't aligned to the left or right. You can usually solve the problem by lining up the left and right edges of all report items and by giving the items the same width.
- Even when you align all items, some columns still get merged in rare cases. Internal unit conversion and rounding during the rendering process can cause the cells to merge. In the report definition language (RDL), you can specify positions and sizes in various units such as inches, pixels, centimeters, and points. Internally, Excel uses points. As a result, inches and centimeters are converted to points during rendering. To minimize conversion operations during rendering, and the potential inaccuracy of rounding, consider specifying all measurements in whole points. An inch is 72 points.

### Report row groups and column groups

Reports that include row groups or column groups contain empty cells when you export them to Excel. The following image shows a report that groups rows on commute distance. Each commute distance can contain more than one customer.

:::image type="content" source="media/exporting-to-microsoft-excel-report-builder-and-ssrs/report-builder-report.png" alt-text="Screenshot that shows a report in the SSRS web portal. Each commute distance row in the report covers multiple customer rows.":::

When you export the report to Excel, the commute distance appears only in one cell of the **Commute Distance** column. When you design the report, you can align the text to the top, middle, or bottom of the row group. That alignment determines whether the value is in the first, middle, or last cell in the exported report. The other cells in that column in the group are empty. The **Name** column, which contains customer names, has no empty cells.

The following image shows the report after your export it to Excel. The empty cells are shaded gray in the image, but that shading isn't part of the exported report.

:::image type="content" source="media/exporting-to-microsoft-excel-report-builder-and-ssrs/excel-report-empty-cells-gray.png" alt-text="Screenshot of an exported report in Excel. Each distance row is first in a range that covers several customer rows. Other cells in the range are gray.":::

After you export a report that contains row groups or column groups to Excel, you need to modify the report before you can display the exported data in a PivotTable. You must add the group value to cells that it's missing from. The worksheet then becomes a flat table with values in all cells. The following image shows the updated worksheet.

:::image type="content" source="media/exporting-to-microsoft-excel-report-builder-and-ssrs/excel-report-no-empty-cells.png" alt-text="Screenshot of an exported report in Excel, with no empty cells. Each row contains a distance value and a name, among other data.":::

If you create a report for the specific purpose of exporting it to Excel for further analysis of the report data, consider not grouping on rows or columns in your report.

## Excel renderer

The following XML code shows the element for the Excel rendering extension in the RSReportServer and RSReportDesigner configuration files:

`<Extension Name="EXCELOPENXML" Type="Microsoft.ReportingServices.Rendering.ExcelOpenXmlRenderer.ExcelOpenXmlRenderer,Microsoft.ReportingServices.ExcelRendering"/>`

The Excel renderer has the following default values and limits:

| Property | Value |
| --- | --- |
| Maximum columns per worksheet | 16,384 |
| Maximum rows per worksheet | 1,048,576 |
| Number of colors allowed in a worksheet | Approximately 16 million (24-bit color) |
| ZIP compressed files | ZIP compression |
| Default font family | Calibri |
| Default font size | 11 points |
| Default row height | 15 points |

Because the report explicitly sets the row height, the default row height affects only rows that are sized automatically during export to Excel.

## Report items in Excel

When you export a report to Excel, the following components are rendered as a range of Excel cells: subreports, rectangles, the report body, and data regions. Text boxes, images, charts, data bars, sparklines, maps, gauges, and indicators are rendered within one Excel cell. But that cell might get merged with other cells. The layout of the rest of the report determines whether merging occurs.

Images, charts, sparklines, data bars, maps, gauges, indicators, and lines are positioned within one Excel cell, but they sit on top of the cell grid. Lines are rendered as cell borders.

Charts, sparklines, data bars, maps, gauges, and indicators are exported as images. The data they depict isn't exported with them. The data isn't available in the Excel workbook unless you include it in a column or row in a data region within a report.

If you want to work with data for charts, sparklines, data bars, maps, gauges, and indicators, you can export the report to a CSV file or generate Atom-compliant data feeds from the report. For more information, see [Export a paginated report to a CSV file (Report Builder)](../../reporting-services/report-builder/exporting-to-a-csv-file-report-builder-and-ssrs.md) and [Generate data feeds from reports (Report Builder)](../../reporting-services/report-builder/generating-data-feeds-from-reports-report-builder-and-ssrs.md).

## Page size

The Excel rendering extension uses the page height and width settings to determine the paper settings of the Excel worksheet. Excel tries to match the `PageHeight` and `PageWidth` property settings to one of the most common paper sizes.

If no matches are found, Excel uses the default page size for the printer. The orientation is set to `Portrait` if the page width is less than the page height. Otherwise, the orientation is set to `Landscape`.

## Worksheet tab names

When you export a report to Excel, page breaks create the report pages, and each page is exported to a different worksheet. If you provide an initial page name for the report, the first worksheet of the Excel workbook has this name. Because each worksheet in a workbook must have a unique name, an integer starting at two and incremented by one is appended to the page name for each worksheet. For example, if the initial page name is **Sales Report by Fiscal Year**, the second worksheet is named **Sales Report by Fiscal Year (2)**. The third one is named **Sales Report by Fiscal Year (3)**, and so on.

If all report pages that are created by page breaks provide new page names, each worksheet has the associated page name. But if these page names aren't unique, the worksheets are named the same way as the initial page names. For example, if the page name of two groups is **Sales for NW**, one worksheet tab has the name **Sales for NW**, and the other **Sales for NW (2)**.

If the report doesn't provide an initial page name or page names for page breaks, the worksheet tabs have the default names **Sheet1**, **Sheet2**, and so on.

SSRS provides properties that you can set for reports, data regions, groups, and rectangles. These properties help you create reports that you can export to Excel in a way that you want. For more information, see [Pagination in paginated reports (Microsoft Report Builder)](../../reporting-services/report-design/pagination-in-reporting-services-report-builder-and-ssrs.md).

## Document properties

The Excel renderer writes the following metadata to the Excel file.

| Report element properties | Description |
| --- | --- |
| Created | Date and time of report execution as an ISO date/time value. |
| Author | Report.Author |
| Description | Report.Description |
| LastSaved | Date and time of report execution as an ISO date/time value. |

## Page headers and footers

The way the page header is rendered depends on the device information `SimplePageHeaders` setting:

- By default, `SimplePageHeaders` is set to **False**. In this case, the header is rendered to the cell grid on the Excel worksheet, at the top of that grid.
- If `SimplePageHeaders` is set to **True**, the header is rendered to the Excel worksheet header section.

The page footer is always rendered to the Excel worksheet footer section, regardless of the value of the `SimplePageHeaders` setting.

Because of Excel limitations, text boxes are the only type of report item that can be rendered in the Excel header and footer sections.

Excel header and footer sections support a maximum of 256 characters, including markup. If this limit is exceeded, the Excel renderer removes markup characters starting at the end of the header or footer string to reduce the number of total characters. If all markup characters are removed and the length still exceeds the maximum, the string is truncated starting from the end.

### SimplePageHeader settings

- When the device information `SimplePageHeaders` setting is set to **False**, the worksheet rows that contain the headers become locked rows. You can freeze or unfreeze the pane in Excel.

- If the Excel settings for printing titles are configured to print these header rows, these headers get printed on every worksheet page except the document map cover sheet.

- In the Report Builder Page Header Properties window:
  - If **Print on first page** isn't selected, the header isn't added to the first report page.
  - If **Print on last page** isn't selected, the header isn't added to the last report page.

## Interactivity

Some interactive elements are supported in Excel. The following sections discuss interactivity.

### Show and hide

There are limitations in the way Excel manages hidden and displayed report items when they're exported. Groups, rows, and columns that contain report items that can expand and collapse are rendered as Excel outlines. But Excel outlines expand and collapse rows and columns across the entire row or column. As a result, report items can be collapsed that aren't intended to be collapsed. Also, Excel's outlining symbols can become cluttered with overlapping outlines.

To address these issues, the Excel rendering extension uses the following outlining rules:

- The report item that can expand and collapse that's closest to the top-left corner can also expand and collapse in Excel. Other report items that share vertical or horizontal space with that top-left item can't expand or collapse in Excel.

- To determine whether a data region is collapsible by rows or columns, the position of two items is taken into account:
  - The report item that controls the visibility
  - The data region that can expand and collapse

  The rules that apply depend on the relative position of these two items:
  - If the item that controls the visibility appears above or below the item that expands and collapses, the item is collapsible by rows.
  - If the item that controls the visibility appears beside the item that expands and collapses, the item is collapsible by columns.
  - If the item that controls the visibility appears the same distance above and beside the item that expands and collapses, the item is collapsible by rows.

- To determine where automatic subtotals are placed in the rendered report, the rendering extension examines the first instance of a dynamic member. If a peer static member appears immediately above it, the dynamic member is assumed to be the subtotal. Outlines are set to indicate that this data is summary data. If there are no static siblings of a dynamic member, the first instance of the instance is the subtotal.

- Due to an Excel limitation, outlines can be nested only up to seven levels.

### Document map

If any document map labels exist in the report, a document map is rendered as an Excel cover worksheet. The worksheet is named **Document map**, and it's located in the first tab position in the workbook.

The `DocumentMapLabel` property of a report item or group determines its label in the document map. Labels are listed in the order that they appear in the report, starting at the first row, in the first column. Each document map label cell is indented the number of levels deep it appears in the report. Each level of indentation is represented by placing the label in a subsequent column. Excel supports up to 256 levels of outline nesting.

The document map outline is rendered as a collapsible Excel outline. The outline structure matches the nested structure of the document map. The expand and collapse state of the outline starts at the second level.

The root node of the map is the report name, or its file name without the .rdl extension. That name isn't interactive.

The renderer uses a 10-point Arial font for the document map links.

### Drillthrough links

A drillthrough link that appears in a text box is rendered as an Excel hyperlink in the cell in which the text is rendered. A drillthrough link for an image or chart is rendered as an Excel hyperlink on the image. When you select a drillthrough link, it opens the client's default browser and goes to the HTML view of the target.

### Hyperlinks

A hyperlink that appears in a text box is rendered as an Excel hyperlink in the cell in which the text is rendered. A hyperlink for an image or chart is rendered as an Excel hyperlink on the image. When you select a hyperlink, it opens the client's default browser and goes to the target URL.

### Interactive sort

In Report Builder, you can select buttons in a report to change the order that tables and matrices display rows and columns in. Excel doesn't support this type of interactive sort.

### Bookmarks

A bookmark link in a text box is rendered as an Excel hyperlink in the cell in which the text is rendered. A bookmark link for an image or chart is rendered as an Excel hyperlink on the image. When you select a bookmark, it goes to the Excel cell in which the bookmarked report item is rendered.

## <a id="ConditionalFormat"></a> Change reports at runtime

In some scenarios, you need a report to render to multiple formats. If it isn't possible to create a report layout that renders the way you want in all required formats, you can use the `RenderFormat` built-in global value. When you use this value, you can conditionally change the report appearance at runtime. This way, you can hide or show report items, depending on the renderer that you use, to get the best results in each format. For more information, see [Built-in Globals and User references in a paginated report (Report Builder)](../../reporting-services/report-design/built-in-collections-built-in-globals-and-users-references-report-builder.md).

## Troubleshoot export to Excel

When you use the virtual service account and execution account, the export to Excel can fail. Specifically, registry key access can be denied.

To get around this issue, you can give read permission to the execution account for the affected registry entry under the virtual user account branch. For example, one possible registry entry is `HKEY_USERS\S-1-5-80-4050220999-2730734961-1537482082-519850261-379003301\Software\Microsoft\Avalon.Graphics`. You then need to restart your computer.

## Related content

- [Rendering behaviors in a paginated report (Report Builder)](../../reporting-services/report-design/rendering-behaviors-report-builder-and-ssrs.md)
- [Interactive functionality - different report rendering extensions](../../reporting-services/report-builder/interactive-functionality-different-report-rendering-extensions.md)
- [Rendering report items in paginated reports (Report Builder)](../../reporting-services/report-design/rendering-report-items-report-builder-and-ssrs.md)
