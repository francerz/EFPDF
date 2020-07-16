EFPDF (Extended Free PDF)
=======================================

This library extends basic functionality of the FPDF class.

FPDF is a PHP class which allows to generate PDF files with pure PHP.
F from FPDF stands for Free: you may use it for any kind of usage and modify it
to suit your needs.

Installation with Composer
---------------------------------------

Add the repository and require to your **composer.json** file:
```json
    {
        "repositories": [{
            "type":"vcs",
            "url":"https://github.com/francerz/EFPDF"
        }],
        "require": {
            "francerz/efpdf": "master"
        }
    }
```

Extended functionality
---------------------------------------

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

### Coordinate Pinning

It's posible to pin coordinates with a name.

```php
// Defines a coordinate pin at current X,Y with name 'start'.
$pdf->SetPin('start');

// Retrieves 'start' pin positions.
$x = $pdf->GetPinX('start');
$y = $pdf->GetPinY('start');

// Moves pdf position back to pin 'start'
$pdf->MoveToPin('start');
```

### Direct UTF-8 decoding text Cell

`CellUTF8($w, $h=0, $txt='', $border=0, $ln=0, $align='', $fill=false, $link='')`

Performs UTF-8 decoding strings without putting right into the code.

```php
$pdf->CellUTF8(0, 10, 'Benjamín pidió una bebida de kiwi y fresa; Noé, sin vergüenza, la más exquisita champaña del menú.');
```

### Right aligned Cell

`CellRight($w, $h=0, $txt='', $border=0, $ln=0, $align='', $fill=false, $link='', $margin=0)`

Puts a cell aligned to the right margin of the page.
Optionally `$margin` can be set to displace the Cell from the right margin.

```php
$pdf->CellRight(15, 5, 'Date: ', 0, 0, 'R', false, '', 30);
$pdf->CellRight(30, 5, date('Y-m-d'), 1, 1, 'C', false, '', 0);
```

> **Note:**  
> `CellRight()` uses `CellUTF8()` and text will be UTF-8 decoded.

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
