# PHP使用PHPExcel生成Excel表格文件

## 前言

由于业务需要,我们需要从业务中汇总数据,并生成Excel文件。
思路是这样的
PHP要导出Excel表格文件->找一个好用的第三方库吧->在Composer的Packages里找一个吧->PHPExcel这么多收藏,就它了!

## PHPExcel

### 概述

>  PHPExcel is a library written in pure PHP and providing a set of classes that allow you to write to and read from different spreadsheet file formats, like Excel (BIFF) .xls, Excel 2007 (OfficeOpenXML) .xlsx, CSV, Libre/OpenOffice Calc .ods, Gnumeric, PDF, HTML, ... This project is built around Microsoft's OpenXML standard and PHP.

由于比较简单,就不翻了。。。

### 安装需求

PHP最低版本5.2.0 
PHP扩展如下

- php_zip
- php_xml
- php_gd2（可选,如果需要精确到列宽,就要安装)

接下来就是检查服务器上的PHP版本和安装扩展

`php -v`

`php -m | grep -E "zip|xml|gd2"`


### 加载第三方类

我们直接采用composer进行安装

`composer require phpoffice/phpexcel`


### 测试demo

我们的业务需求是生成excel文件,也就是扩展名为`xls`的文件,所以我们定位到的是项目给的事例中的`Examples/14excel5.php`

下面是事例源码,我会在适当的注释。

```php

<?php
/**
 * PHPExcel
 *
 * Copyright (c) 2006 - 2015 PHPExcel
 *
 * This library is free software; you can redistribute it and/or
 * modify it under the terms of the GNU Lesser General Public
 * License as published by the Free Software Foundation; either
 * version 2.1 of the License, or (at your option) any later version.
 *
 * This library is distributed in the hope that it will be useful,
 * but WITHOUT ANY WARRANTY; without even the implied warranty of
 * MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the GNU
 * Lesser General Public License for more details.
 *
 * You should have received a copy of the GNU Lesser General Public
 * License along with this library; if not, write to the Free Software
 * Foundation, Inc., 51 Franklin Street, Fifth Floor, Boston, MA  02110-1301  USA
 *
 * @category   PHPExcel
 * @package    PHPExcel
 * @copyright  Copyright (c) 2006 - 2015 PHPExcel (http://www.codeplex.com/PHPExcel)
 * @license    http://www.gnu.org/licenses/old-licenses/lgpl-2.1.txt	LGPL
 * @version    ##VERSION##, ##DATE##
 */

/** Error reporting */
error_reporting(E_ALL);
ini_set('display_errors', TRUE);
ini_set('display_startup_errors', TRUE);

define('EOL',(PHP_SAPI == 'cli') ? PHP_EOL : '<br />');

date_default_timezone_set('Europe/London'); //设置时区,我们根据自己项目所处的位置设置就好,不用跟本例一致

include "05featuredemo.inc.php"; //引入其他文件,在这个文件里进行了文件内容的填充

/** PHPExcel_IOFactory */
require_once dirname(__FILE__) . '/../Classes/PHPExcel/IOFactory.php'; //引入数据流IO工厂类文件


// Save Excel 95 file
echo date('H:i:s') , " Write to Excel5 format" , EOL;
$callStartTime = microtime(true);

$objWriter = PHPExcel_IOFactory::createWriter($objPHPExcel, 'Excel5'); //创建一个文件写入对象
$objWriter->save(str_replace('.php', '.xls', __FILE__)); //写入文件
$callEndTime = microtime(true);
$callTime = $callEndTime - $callStartTime; //计算耗时

echo date('H:i:s') , " File written to " , str_replace('.php', '.xls', pathinfo(__FILE__, PATHINFO_BASENAME)) , EOL;
echo 'Call time to write Workbook was ' , sprintf('%.4f',$callTime) , " seconds" , EOL;
// Echo memory usage
echo date('H:i:s') , ' Current memory usage: ' , (memory_get_usage(true) / 1024 / 1024) , " MB" , EOL;


// Echo memory peak usage
echo date('H:i:s') , " Peak memory usage: " , (memory_get_peak_usage(true) / 1024 / 1024) , " MB" , EOL;

// Echo done
echo date('H:i:s') , " Done writing file" , EOL;
echo 'File has been created in ' , getcwd() , EOL;


```

所以上面的文件的作用是对做好的文件进行格式转换存储,下面继续看`05featuredemo.inc.php`.

```php

<?php
/**
 * PHPExcel
 *
 * Copyright (c) 2006 - 2015 PHPExcel
 *
 * This library is free software; you can redistribute it and/or
 * modify it under the terms of the GNU Lesser General Public
 * License as published by the Free Software Foundation; either
 * version 2.1 of the License, or (at your option) any later version.
 *
 * This library is distributed in the hope that it will be useful,
 * but WITHOUT ANY WARRANTY; without even the implied warranty of
 * MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the GNU
 * Lesser General Public License for more details.
 *
 * You should have received a copy of the GNU Lesser General Public
 * License along with this library; if not, write to the Free Software
 * Foundation, Inc., 51 Franklin Street, Fifth Floor, Boston, MA  02110-1301  USA
 *
 * @category   PHPExcel
 * @package    PHPExcel
 * @copyright  Copyright (c) 2006 - 2015 PHPExcel (http://www.codeplex.com/PHPExcel)
 * @license    http://www.gnu.org/licenses/old-licenses/lgpl-2.1.txt	LGPL
 * @version    ##VERSION##, ##DATE##
 */

/** Error reporting */
error_reporting(E_ALL);

/** Include PHPExcel */
require_once dirname(__FILE__) . '/../Classes/PHPExcel.php'; //引入核心库


// Create new PHPExcel object
echo date('H:i:s') , " Create new PHPExcel object" , EOL;
$objPHPExcel = new PHPExcel(); //创建一个PHPExcel对象

// Set document properties 设置文件信息
echo date('H:i:s') , " Set document properties" , EOL;
$objPHPExcel->getProperties()->setCreator("Maarten Balliauw")
							 ->setLastModifiedBy("Maarten Balliauw")
							 ->setTitle("Office 2007 XLSX Test Document")
							 ->setSubject("Office 2007 XLSX Test Document")
							 ->setDescription("Test document for Office 2007 XLSX, generated using PHP classes.")
							 ->setKeywords("office 2007 openxml php")
							 ->setCategory("Test result file");


// Create a first sheet, representing sales data
echo date('H:i:s') , " Add some data" , EOL;
$objPHPExcel->setActiveSheetIndex(0);
$objPHPExcel->getActiveSheet()->setCellValue('B1', 'Invoice');
$objPHPExcel->getActiveSheet()->setCellValue('D1', PHPExcel_Shared_Date::PHPToExcel( gmmktime(0,0,0,date('m'),date('d'),date('Y')) ));//设置具体的数值
$objPHPExcel->getActiveSheet()->getStyle('D1')->getNumberFormat()->setFormatCode(PHPExcel_Style_NumberFormat::FORMAT_DATE_XLSX15); //设置展示格式
$objPHPExcel->getActiveSheet()->setCellValue('E1', '#12566');

$objPHPExcel->getActiveSheet()->setCellValue('A3', 'Product Id');
$objPHPExcel->getActiveSheet()->setCellValue('B3', 'Description');
$objPHPExcel->getActiveSheet()->setCellValue('C3', 'Price');
$objPHPExcel->getActiveSheet()->setCellValue('D3', 'Amount');
$objPHPExcel->getActiveSheet()->setCellValue('E3', 'Total');

$objPHPExcel->getActiveSheet()->setCellValue('A4', '1001');
$objPHPExcel->getActiveSheet()->setCellValue('B4', 'PHP for dummies');
$objPHPExcel->getActiveSheet()->setCellValue('C4', '20');
$objPHPExcel->getActiveSheet()->setCellValue('D4', '1');
$objPHPExcel->getActiveSheet()->setCellValue('E4', '=IF(D4<>"",C4*D4,"")'); //支持直接写入公式

$objPHPExcel->getActiveSheet()->setCellValue('A5', '1012');
$objPHPExcel->getActiveSheet()->setCellValue('B5', 'OpenXML for dummies');
$objPHPExcel->getActiveSheet()->setCellValue('C5', '22');
$objPHPExcel->getActiveSheet()->setCellValue('D5', '2');
$objPHPExcel->getActiveSheet()->setCellValue('E5', '=IF(D5<>"",C5*D5,"")');

$objPHPExcel->getActiveSheet()->setCellValue('E6', '=IF(D6<>"",C6*D6,"")');
$objPHPExcel->getActiveSheet()->setCellValue('E7', '=IF(D7<>"",C7*D7,"")');
$objPHPExcel->getActiveSheet()->setCellValue('E8', '=IF(D8<>"",C8*D8,"")');
$objPHPExcel->getActiveSheet()->setCellValue('E9', '=IF(D9<>"",C9*D9,"")');

$objPHPExcel->getActiveSheet()->setCellValue('D11', 'Total excl.:');
$objPHPExcel->getActiveSheet()->setCellValue('E11', '=SUM(E4:E9)'); //支持直接写入Excel函数

$objPHPExcel->getActiveSheet()->setCellValue('D12', 'VAT:');
$objPHPExcel->getActiveSheet()->setCellValue('E12', '=E11*0.21');

$objPHPExcel->getActiveSheet()->setCellValue('D13', 'Total incl.:');
$objPHPExcel->getActiveSheet()->setCellValue('E13', '=E11+E12');

// Add comment
echo date('H:i:s') , " Add comments" , EOL;

$objPHPExcel->getActiveSheet()->getComment('E11')->setAuthor('PHPExcel');
$objCommentRichText = $objPHPExcel->getActiveSheet()->getComment('E11')->getText()->createTextRun('PHPExcel:');
$objCommentRichText->getFont()->setBold(true);
$objPHPExcel->getActiveSheet()->getComment('E11')->getText()->createTextRun("\r\n");
$objPHPExcel->getActiveSheet()->getComment('E11')->getText()->createTextRun('Total amount on the current invoice, excluding VAT.');

$objPHPExcel->getActiveSheet()->getComment('E12')->setAuthor('PHPExcel');
$objCommentRichText = $objPHPExcel->getActiveSheet()->getComment('E12')->getText()->createTextRun('PHPExcel:');
$objCommentRichText->getFont()->setBold(true);
$objPHPExcel->getActiveSheet()->getComment('E12')->getText()->createTextRun("\r\n");
$objPHPExcel->getActiveSheet()->getComment('E12')->getText()->createTextRun('Total amount of VAT on the current invoice.');

$objPHPExcel->getActiveSheet()->getComment('E13')->setAuthor('PHPExcel');
$objCommentRichText = $objPHPExcel->getActiveSheet()->getComment('E13')->getText()->createTextRun('PHPExcel:');
$objCommentRichText->getFont()->setBold(true);
$objPHPExcel->getActiveSheet()->getComment('E13')->getText()->createTextRun("\r\n");
$objPHPExcel->getActiveSheet()->getComment('E13')->getText()->createTextRun('Total amount on the current invoice, including VAT.');
$objPHPExcel->getActiveSheet()->getComment('E13')->setWidth('100pt');
$objPHPExcel->getActiveSheet()->getComment('E13')->setHeight('100pt');
$objPHPExcel->getActiveSheet()->getComment('E13')->setMarginLeft('150pt');
$objPHPExcel->getActiveSheet()->getComment('E13')->getFillColor()->setRGB('EEEEEE');


// Add rich-text string
echo date('H:i:s') , " Add rich-text string" , EOL;
$objRichText = new PHPExcel_RichText(); //创建富文本对象
$objRichText->createText('This invoice is ');

$objPayable = $objRichText->createTextRun('payable within thirty days after the end of the month');
$objPayable->getFont()->setBold(true);
$objPayable->getFont()->setItalic(true);
$objPayable->getFont()->setColor( new PHPExcel_Style_Color( PHPExcel_Style_Color::COLOR_DARKGREEN ) );

$objRichText->createText(', unless specified otherwise on the invoice.');

$objPHPExcel->getActiveSheet()->getCell('A18')->setValue($objRichText);

// Merge cells
echo date('H:i:s') , " Merge cells" , EOL;
$objPHPExcel->getActiveSheet()->mergeCells('A18:E22');
$objPHPExcel->getActiveSheet()->mergeCells('A28:B28');		// Just to test...
$objPHPExcel->getActiveSheet()->unmergeCells('A28:B28');	// Just to test...

// Protect cells
echo date('H:i:s') , " Protect cells" , EOL;
$objPHPExcel->getActiveSheet()->getProtection()->setSheet(true);	// Needs to be set to true in order to enable any worksheet protection!
$objPHPExcel->getActiveSheet()->protectCells('A3:E13', 'PHPExcel');

// Set cell number formats
echo date('H:i:s') , " Set cell number formats" , EOL;
$objPHPExcel->getActiveSheet()->getStyle('E4:E13')->getNumberFormat()->setFormatCode(PHPExcel_Style_NumberFormat::FORMAT_CURRENCY_EUR_SIMPLE);

// Set column widths
echo date('H:i:s') , " Set column widths" , EOL;
$objPHPExcel->getActiveSheet()->getColumnDimension('B')->setAutoSize(true);
$objPHPExcel->getActiveSheet()->getColumnDimension('D')->setWidth(12);
$objPHPExcel->getActiveSheet()->getColumnDimension('E')->setWidth(12);

// Set fonts
echo date('H:i:s') , " Set fonts" , EOL;
$objPHPExcel->getActiveSheet()->getStyle('B1')->getFont()->setName('Candara');
$objPHPExcel->getActiveSheet()->getStyle('B1')->getFont()->setSize(20);
$objPHPExcel->getActiveSheet()->getStyle('B1')->getFont()->setBold(true);
$objPHPExcel->getActiveSheet()->getStyle('B1')->getFont()->setUnderline(PHPExcel_Style_Font::UNDERLINE_SINGLE);
$objPHPExcel->getActiveSheet()->getStyle('B1')->getFont()->getColor()->setARGB(PHPExcel_Style_Color::COLOR_WHITE);

$objPHPExcel->getActiveSheet()->getStyle('D1')->getFont()->getColor()->setARGB(PHPExcel_Style_Color::COLOR_WHITE);
$objPHPExcel->getActiveSheet()->getStyle('E1')->getFont()->getColor()->setARGB(PHPExcel_Style_Color::COLOR_WHITE);

$objPHPExcel->getActiveSheet()->getStyle('D13')->getFont()->setBold(true);
$objPHPExcel->getActiveSheet()->getStyle('E13')->getFont()->setBold(true);

// Set alignments
echo date('H:i:s') , " Set alignments" , EOL;
$objPHPExcel->getActiveSheet()->getStyle('D11')->getAlignment()->setHorizontal(PHPExcel_Style_Alignment::HORIZONTAL_RIGHT);
$objPHPExcel->getActiveSheet()->getStyle('D12')->getAlignment()->setHorizontal(PHPExcel_Style_Alignment::HORIZONTAL_RIGHT);
$objPHPExcel->getActiveSheet()->getStyle('D13')->getAlignment()->setHorizontal(PHPExcel_Style_Alignment::HORIZONTAL_RIGHT);

$objPHPExcel->getActiveSheet()->getStyle('A18')->getAlignment()->setHorizontal(PHPExcel_Style_Alignment::HORIZONTAL_JUSTIFY);
$objPHPExcel->getActiveSheet()->getStyle('A18')->getAlignment()->setVertical(PHPExcel_Style_Alignment::VERTICAL_CENTER);

$objPHPExcel->getActiveSheet()->getStyle('B5')->getAlignment()->setShrinkToFit(true);

// Set thin black border outline around column  设置复杂的格式
echo date('H:i:s') , " Set thin black border outline around column" , EOL;
$styleThinBlackBorderOutline = array(
	'borders' => array(
		'outline' => array(
			'style' => PHPExcel_Style_Border::BORDER_THIN,
			'color' => array('argb' => 'FF000000'),
		),
	),
);
$objPHPExcel->getActiveSheet()->getStyle('A4:E10')->applyFromArray($styleThinBlackBorderOutline);


// Set thick brown border outline around "Total"
echo date('H:i:s') , " Set thick brown border outline around Total" , EOL;
$styleThickBrownBorderOutline = array(
	'borders' => array(
		'outline' => array(
			'style' => PHPExcel_Style_Border::BORDER_THICK,
			'color' => array('argb' => 'FF993300'),
		),
	),
);
$objPHPExcel->getActiveSheet()->getStyle('D13:E13')->applyFromArray($styleThickBrownBorderOutline);

// Set fills
echo date('H:i:s') , " Set fills" , EOL;
$objPHPExcel->getActiveSheet()->getStyle('A1:E1')->getFill()->setFillType(PHPExcel_Style_Fill::FILL_SOLID);
$objPHPExcel->getActiveSheet()->getStyle('A1:E1')->getFill()->getStartColor()->setARGB('FF808080');

// Set style for header row using alternative method
echo date('H:i:s') , " Set style for header row using alternative method" , EOL;
$objPHPExcel->getActiveSheet()->getStyle('A3:E3')->applyFromArray(
		array(
			'font'    => array(
				'bold'      => true
			),
			'alignment' => array(
				'horizontal' => PHPExcel_Style_Alignment::HORIZONTAL_RIGHT,
			),
			'borders' => array(
				'top'     => array(
 					'style' => PHPExcel_Style_Border::BORDER_THIN
 				)
			),
			'fill' => array(
	 			'type'       => PHPExcel_Style_Fill::FILL_GRADIENT_LINEAR,
	  			'rotation'   => 90,
	 			'startcolor' => array(
	 				'argb' => 'FFA0A0A0'
	 			),
	 			'endcolor'   => array(
	 				'argb' => 'FFFFFFFF'
	 			)
	 		)
		)
);

$objPHPExcel->getActiveSheet()->getStyle('A3')->applyFromArray(
		array(
			'alignment' => array(
				'horizontal' => PHPExcel_Style_Alignment::HORIZONTAL_LEFT,
			),
			'borders' => array(
				'left'     => array(
 					'style' => PHPExcel_Style_Border::BORDER_THIN
 				)
			)
		)
);

$objPHPExcel->getActiveSheet()->getStyle('B3')->applyFromArray(
		array(
			'alignment' => array(
				'horizontal' => PHPExcel_Style_Alignment::HORIZONTAL_LEFT,
			)
		)
);

$objPHPExcel->getActiveSheet()->getStyle('E3')->applyFromArray(
		array(
			'borders' => array(
				'right'     => array(
 					'style' => PHPExcel_Style_Border::BORDER_THIN
 				)
			)
		)
);

// Unprotect a cell
echo date('H:i:s') , " Unprotect a cell" , EOL;
$objPHPExcel->getActiveSheet()->getStyle('B1')->getProtection()->setLocked(PHPExcel_Style_Protection::PROTECTION_UNPROTECTED);

// Add a hyperlink to the sheet
echo date('H:i:s') , " Add a hyperlink to an external website" , EOL;
$objPHPExcel->getActiveSheet()->setCellValue('E26', 'www.phpexcel.net');
$objPHPExcel->getActiveSheet()->getCell('E26')->getHyperlink()->setUrl('http://www.phpexcel.net'); /加入超链接
$objPHPExcel->getActiveSheet()->getCell('E26')->getHyperlink()->setTooltip('Navigate to website');
$objPHPExcel->getActiveSheet()->getStyle('E26')->getAlignment()->setHorizontal(PHPExcel_Style_Alignment::HORIZONTAL_RIGHT);

echo date('H:i:s') , " Add a hyperlink to another cell on a different worksheet within the workbook" , EOL;
$objPHPExcel->getActiveSheet()->setCellValue('E27', 'Terms and conditions');
$objPHPExcel->getActiveSheet()->getCell('E27')->getHyperlink()->setUrl("sheet://'Terms and conditions'!A1");
$objPHPExcel->getActiveSheet()->getCell('E27')->getHyperlink()->setTooltip('Review terms and conditions');
$objPHPExcel->getActiveSheet()->getStyle('E27')->getAlignment()->setHorizontal(PHPExcel_Style_Alignment::HORIZONTAL_RIGHT);

// Add a drawing to the worksheet 
echo date('H:i:s') , " Add a drawing to the worksheet" , EOL;
$objDrawing = new PHPExcel_Worksheet_Drawing(); //添加图片
$objDrawing->setName('Logo');
$objDrawing->setDescription('Logo');
$objDrawing->setPath('./images/officelogo.jpg');
$objDrawing->setHeight(36);
$objDrawing->setWorksheet($objPHPExcel->getActiveSheet());

// Add a drawing to the worksheet
echo date('H:i:s') , " Add a drawing to the worksheet" , EOL;
$objDrawing = new PHPExcel_Worksheet_Drawing();
$objDrawing->setName('Paid');
$objDrawing->setDescription('Paid');
$objDrawing->setPath('./images/paid.png');
$objDrawing->setCoordinates('B15');
$objDrawing->setOffsetX(110);
$objDrawing->setRotation(25);
$objDrawing->getShadow()->setVisible(true);
$objDrawing->getShadow()->setDirection(45);
$objDrawing->setWorksheet($objPHPExcel->getActiveSheet());

// Add a drawing to the worksheet
echo date('H:i:s') , " Add a drawing to the worksheet" , EOL;
$objDrawing = new PHPExcel_Worksheet_Drawing();
$objDrawing->setName('PHPExcel logo');
$objDrawing->setDescription('PHPExcel logo');
$objDrawing->setPath('./images/phpexcel_logo.gif');
$objDrawing->setHeight(36);
$objDrawing->setCoordinates('D24');
$objDrawing->setOffsetX(10);
$objDrawing->setWorksheet($objPHPExcel->getActiveSheet());

// Play around with inserting and removing rows and columns
echo date('H:i:s') , " Play around with inserting and removing rows and columns" , EOL;
$objPHPExcel->getActiveSheet()->insertNewRowBefore(6, 10);
$objPHPExcel->getActiveSheet()->removeRow(6, 10);
$objPHPExcel->getActiveSheet()->insertNewColumnBefore('E', 5);
$objPHPExcel->getActiveSheet()->removeColumn('E', 5);

// Set header and footer. When no different headers for odd/even are used, odd header is assumed.
echo date('H:i:s') , " Set header/footer" , EOL;
$objPHPExcel->getActiveSheet()->getHeaderFooter()->setOddHeader('&L&BInvoice&RPrinted on &D');
$objPHPExcel->getActiveSheet()->getHeaderFooter()->setOddFooter('&L&B' . $objPHPExcel->getProperties()->getTitle() . '&RPage &P of &N');

// Set page orientation and size
echo date('H:i:s') , " Set page orientation and size" , EOL;
$objPHPExcel->getActiveSheet()->getPageSetup()->setOrientation(PHPExcel_Worksheet_PageSetup::ORIENTATION_PORTRAIT);
$objPHPExcel->getActiveSheet()->getPageSetup()->setPaperSize(PHPExcel_Worksheet_PageSetup::PAPERSIZE_A4);

// Rename first worksheet
echo date('H:i:s') , " Rename first worksheet" , EOL;
$objPHPExcel->getActiveSheet()->setTitle('Invoice');


// Create a new worksheet, after the default sheet
echo date('H:i:s') , " Create a second Worksheet object" , EOL;
$objPHPExcel->createSheet();

// Llorem ipsum... 很长的文本
$sLloremIpsum = 'Lorem ipsum dolor sit amet, consectetuer adipiscing elit. Vivamus eget ante. Sed cursus nunc semper tortor. Aliquam luctus purus non elit. Fusce vel elit commodo sapien dignissim dignissim. Pellentesque habitant morbi tristique senectus et netus et malesuada fames ac turpis egestas. Curabitur accumsan magna sed massa. Nullam bibendum quam ac ipsum. Lorem ipsum dolor sit amet, consectetuer adipiscing elit. Proin augue. Praesent malesuada justo sed orci. Pellentesque lacus ligula, sodales quis, ultricies a, ultricies vitae, elit. Sed luctus consectetuer dolor. Vivamus vel sem ut nisi sodales accumsan. Nunc et felis. Suspendisse semper viverra odio. Morbi at odio. Integer a orci a purus venenatis molestie. Nam mattis. Praesent rhoncus, nisi vel mattis auctor, neque nisi faucibus sem, non dapibus elit pede ac nisl. Cras turpis.';

// Add some data to the second sheet, resembling some different data types
echo date('H:i:s') , " Add some data" , EOL;
$objPHPExcel->setActiveSheetIndex(1);
$objPHPExcel->getActiveSheet()->setCellValue('A1', 'Terms and conditions');
$objPHPExcel->getActiveSheet()->setCellValue('A3', $sLloremIpsum);
$objPHPExcel->getActiveSheet()->setCellValue('A4', $sLloremIpsum);
$objPHPExcel->getActiveSheet()->setCellValue('A5', $sLloremIpsum);
$objPHPExcel->getActiveSheet()->setCellValue('A6', $sLloremIpsum);

// Set the worksheet tab color
echo date('H:i:s') , " Set the worksheet tab color" , EOL;
$objPHPExcel->getActiveSheet()->getTabColor()->setARGB('FF0094FF');;

// Set alignments
echo date('H:i:s') , " Set alignments" , EOL;
$objPHPExcel->getActiveSheet()->getStyle('A3:A6')->getAlignment()->setWrapText(true);

// Set column widths
echo date('H:i:s') , " Set column widths" , EOL;
$objPHPExcel->getActiveSheet()->getColumnDimension('A')->setWidth(80);

// Set fonts
echo date('H:i:s') , " Set fonts" , EOL;
$objPHPExcel->getActiveSheet()->getStyle('A1')->getFont()->setName('Candara');
$objPHPExcel->getActiveSheet()->getStyle('A1')->getFont()->setSize(20);
$objPHPExcel->getActiveSheet()->getStyle('A1')->getFont()->setBold(true);
$objPHPExcel->getActiveSheet()->getStyle('A1')->getFont()->setUnderline(PHPExcel_Style_Font::UNDERLINE_SINGLE);

$objPHPExcel->getActiveSheet()->getStyle('A3:A6')->getFont()->setSize(8);

// Add a drawing to the worksheet
echo date('H:i:s') , " Add a drawing to the worksheet" , EOL;
$objDrawing = new PHPExcel_Worksheet_Drawing();
$objDrawing->setName('Terms and conditions');
$objDrawing->setDescription('Terms and conditions');
$objDrawing->setPath('./images/termsconditions.jpg');
$objDrawing->setCoordinates('B14');
$objDrawing->setWorksheet($objPHPExcel->getActiveSheet());

// Set page orientation and size
echo date('H:i:s') , " Set page orientation and size" , EOL;
$objPHPExcel->getActiveSheet()->getPageSetup()->setOrientation(PHPExcel_Worksheet_PageSetup::ORIENTATION_LANDSCAPE);
$objPHPExcel->getActiveSheet()->getPageSetup()->setPaperSize(PHPExcel_Worksheet_PageSetup::PAPERSIZE_A4);

// Rename second worksheet
echo date('H:i:s') , " Rename second worksheet" , EOL;
$objPHPExcel->getActiveSheet()->setTitle('Terms and conditions');


// Set active sheet index to the first sheet, so Excel opens this as the first sheet
$objPHPExcel->setActiveSheetIndex(0);


```

根据上面的事例,我们想要的效果都能实现了。 :)


### 写一个例子

不过要注意的是,**demo上实现了功能,并不带代表着代码粘过来就能用**。
别忘了我们使用composer加载的,demo是直接加载的文件,细微的差别需要在实践上进一步体会。

最后完成的效果是这样的




## 参考资料

- https://packagist.org/packages/phpoffice/phpexcel
- https://github.com/PHPOffice/PHPExcel