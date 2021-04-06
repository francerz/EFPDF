EFPDF (Extended Free PDF)
=======================================

[![Packagist](https://img.shields.io/packagist/vpre/francerz/efpdf)](https://packagist.org/packages/francerz/efpdf)
[![Build Status](https://github.com/francerz/efpdf/workflows/PHP%20Composer/badge.svg?branch=master)](https://github.com/francerz/EFPDF/actions?query=workflow%3A%22PHP+Composer%22+branch%3Amaster)

This library extends basic functionality of the FPDF class.

FPDF is a PHP class which allows to generate PDF files with pure PHP.
F from FPDF stands for Free: you may use it for any kind of usage and modify it
to suit your needs.

Installation
---------------------------------------

The easiest and optimal way to install EFPDF is by using [Composer](https://getcomposer.org).

If you already using Composer, just run the next command and will be fully
installed.

```bash
composer require francerz/EFPDF
```

Extended functionality
---------------------------------------

### Output result as PSR-7 ResponseInterface

The EFPDF allows to output resultant PDF to PSR-7 compliant ResponseInterface
object. Relies on PSR-17 Http Factories to create the instances.

```php
OutputPsr7(
  \Psr\Http\Message\ResponseFactoryInterface $responseFactory,
  \Psr\Http\Messsage\StreamFactoryInterface $streamFactory,
  string $filename,
  bool $inline = true,
  bool $isUTF8 = false
)
```

Usage example
```php
$pdf = new EFPDF();
// ... create PDF

// ... load HTTP Factories
$responseFactory = new ResponseFactory(); // from a PSR-17 compliant library
$streamFactory = new StreamFactory(); // from a PSR-17 compliant library

// Output PSR-7 file
$response = $pdf->OutputPsr7($responseFactory, $streamFactory, 'my-file.pdf');
```

#### Alternative simplified method `OutputPsr7WithManager`

Alternatively you can use HttpFactoryManager to handle multiple factories
instances to reduce parameters, improving reusability.

```php
OutputPsr7WithManager(
  \Francerz\Http\Utils\HttpFactoryManager $hfm,
  string $filename,
  bool $inline = true,
  bool $isUTF8 = false
)
```

Usage example
```php
$pdf = new EFPDF();
// ... create PDF

// Output PSR-7 Response object
$factories = new HttpFactoryManager(new ResponseFactory(), new StreamFactory());
$response = $pdf->OutputPsr7WithManager($factories, 'my-file.pdf');
```

##### Even simplier with in-home HTTP library

In the following example is used the simplified version with in-home HTTP
library [francerz/http](https://packagist.org/packages/francerz/http) wich
is PSR-7, PSR-17 and PSR-18 compliant.

```php
$pdf = new EFPDF();
// ... create PDF

// Output PSR-7 Response object
$response = $pdf->OutputPsr7WithManager(HttpFactory::getManager(), 'my-file.pdf');
```



### Relative Positioning and Sizing

Its allowed to use X, Y, Width and Height as percents of current page size.

```php
// Sets Y position at 25% (one quarter) from page top.
$pdf->SetY('25%');
// Sets X position at 25% (one quarter) from page left.
$pdf->SetX('25%');

// Draws a cell with 50% width and height of current page size.
$pdf->Cell('50%','50%', '', 1);
```

Also the positioning and sizing can be relative to the page content area,
inside the margins.

```php
// Sets Y position at top margin.
$pdf->SetY('~0');
// Sets X position at left margin.
$pdf->SetX('~0');

// Draws a cell with 25% width and 10% height of current page content.
$pdf->Cell('~25%', '~10%', '', 1);
```

Therefore, you can get measure calculations with methods `CalcX($x)`, `CalcY($y)`,
`CalcWidth($w)` and `CalcHeight($h)`.

### Offset positioning

Allows to increase or decrease current position.

```php
// translates pointer 10 units right to current X.
$pdf->OffsetX(10);

// translates pointer 20 units bottom to current Y.
$pdf->OffsetY(20);

// translates equivalent to two previous in a single line.
$pdf->OffsetXY(10, 20);

// offset may be negative and relative units
$pdf->OffsetXY(-10, '~10%');
```

### Coordinate Pinning

It's posible to pin coordinates with a name.

```php
MoveToPin($pinName, $axis = 'XY', $offset = 0, $offsetY = 0);
```

```php
// Defines a coordinate pin at current X,Y with name 'start'.
$pdf->SetPin('start');

// Retrieves 'start' pin positions.
$x = $pdf->GetPinX('start');
$y = $pdf->GetPinY('start');

// Moves pdf position back to pin 'start'
$pdf->MoveToPin('start');

// Moves pdf X position back to pin 'start'
$pdf->MoveToPin('start','X');

// Moves pdf Y position back to pin 'start'
$pdf->MoveToPin('start','Y');

// Moves pdf X position back to pin 'start' and adds 10 units.
$pdf->MoveToPin('start', 'X', 10);

// Moves pdf Y position back to pin 'start' and adds 20 units.
$pdf->ModeToPin('start', 'Y', 20);

// Moves pdf position back to pin 'start' and adds X: 10 units, Y: 20 units.
$pdf->MoveToPin('start', 'XY', 10, 20);
```

### Relative Cell Height based on Font Size

```php
SetLineHeight(float $size)
```

Define automatic line height size based on current Font Size.

```php
$pdf->SetFontSize(10);
$pdf->SetLineHeight(1.1); // 110% of actual size (11pt)

$pdf->Cell('100%', null, 'This text is size 10pt, but Cell is 11pt height with LineHeight 1.1');
```

### Simplified content encoding

```php
SetSourceEncoding(string $encoding)
```

Allows internal decoding strings without writing on each cell.

```php
$pdf->SetSourceEncoding('UTF-8');
$pdf->Cell('100%', null, 'Benjamín pidió una bebida de kiwi y fresa; Noé, sin vergüenza, la más exquisita champaña del menú.');
```

### Right aligned Cell

```php
CellRight($w, $h=0, $txt='', $border=0, $ln=0, $align='', $fill=false, $link='', $margin=0)
```

Puts a cell aligned to the right margin of the page.
Optionally `$margin` can be set to displace the Cell from the right margin.

```php
$pdf->CellRight(15, 5, 'Date: ', 0, 0, 'R', false, '', 30);
$pdf->CellRight(30, 5, date('Y-m-d'), 1, 1, 'C', false, '', 0);
```

### Header and Footer

Allows to define Header and Footer using anonymous functions.

* `SetHeader(callable $headerFunc, $headerHeight = null)`  
  Sets a callable function that will execute when Header is loading.
  If parameter `$headerHeight` is null, then will be calculated and body content
  will be displaced. If is set, then content will be displaced `$headerHeight`
  plus the top margin.
* `SetFooter(callable $footerFunc, $footerHeight = null)`  
  Sets a callable function that will execute when Footer is loading.
  If parameter `$footerHeight` is null, then will be calculated and page break
  will executed before reaching footer content. If is set, the footer will
  be displaced from based on page bottom edge.

```php
$userName = "My Name";

$pdf = new EFPDF();
$pdf->AliasNbPages();
$pdf->SetFont('Arial', '', 12);
$pdf->SetHeader(function(EFPDF $efpdf) use ($userName) {
    $efpdf->Cell('~100%', 10, "Hello {$userName}", 1);
});
$pdf->SetFooter(function(EFPDF $efpdf) {
    $efpdf->Cell('~100%', 10, 'Page '.$efpdf->PageNo(), 1, 0, 'R');
});
$pdf->AddPage();

$pdf->Output('I');
```
> **Note:**  
> It's important to invoke `SetHeader()` and `SetFooter()` before `AddPage()`.

### Barcode support

```php
$pdf->barcode128(string $code, $w, $h, $x = '0+', $y = '0+');
```

Using the `barcode128` puts the given `$code` ASCII string at the given `$x`
and `$y` position. And with given `$w` (width) and `$h` (height). This measures
are compatible with the relative positioning and sizing.

If no `$x` or `$y` is set, then will be the current PDF position.

```php
$pdf->barcode39(string $code, $w, $h, $x, $y);
```

Using the `barcode39` puts the given `$code` (`[-0-9A-Z. *$/+%]`) string at the
given `$x` and `$y` position, with the given `$w` (width) and `$h` (height).
This meaures are compatible with the relative positioning and sizing.
