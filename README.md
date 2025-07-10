# FMX Skia Tips

A list of tips for users of skia and FMX

1. How to draw a line or bezier curve witha rounded end
2. How to create a dotted line
3. How to draw a bezier curve using skia
4. How to draw circles
5. How to clear the canvas
6. How to draw a rounded rectangle
7. How to draw some Text
8. How to draw a simple linear gradient
9. How to draw an oval
10. How to draw arcs
11. How to load an image
12. How to create a fill of lines hatch horizontal lines
13. How to save a canvas drawing to a pdf file

**1. How to draw a line or bezier curve witha rounded end?**
   
To draw a line with a rounded end in skia you need to change the StrokeCap to Round. Here is a simple example using Delphi:

    procedure drawRoundedLine (ACanvas : ISkCanvas; x1, y1, x2, y2 : single);
    var LPaint : ISkPaint;
    begin 
      LPaint := TSkPaint.Create;
      LPaint.AntiAlias := True;
      LPaint.Color := TAlphaColors.Blue;
      LPaint.StrokeWidth := 2;
      LPaint.Style := TSkPaintStyle.Stroke;
      LPaint.StrokeCap := TSkStrokeCap.Round; 

      ACanvas.DrawLine(x1, y1, x2, y2, LPaint);
    end;

**2. How to create a dotted line?**

It's fairly straightforward to create a dotted line. It relies on using PathEffect and the method MakeDash.

The following example uses a TSkPaintBox which is the skia enabled equivalent of the usual TPaintBox in Delphi.

The TSkPaintBox has a OnDraw event which you can use to add your drawing commands. It's common practice to surround the drawing code with a save and restore. This article explains the reasoning in more detail: https://stackoverflow.com/questions/59650399/what-is-canvas-save-and-canvas-restore.

The complete code for the OnDraw event is shown below:

    procedure TfrmMain.SkPaintBox1Draw(ASender: TObject; const ACanvas: ISkCanvas;
    const ADest: TRectF; const AOpacity: Single);
     var LPaint :ISkPaint;
     LDash : TArray<single>;
    begin
      ACanvas.Save;
      try
        LPaint := TSkPaint.Create (TSKPaintStyle.Stroke);
        LPaint.AntiAlias := True;

        LPaint.StrokeCap   :=  TSkStrokeCap.Round; // Butt, Round, Square
        LPaint.StrokeJoin  := TSkStrokeJoin.Round; // Miter, Round, Bevel
        LPaint.StrokeWidth := 3;
        LPaint.Color := TAlphaColors.Blue;
        SetLength (LDash, 4);
        // Set up the dash pattern
        LDash[0] := 8;
        LDash[1] := 8;
        LDash[2] := 4;
        LDash[3] := 8;

        LPaint.PathEffect := TSkPathEffect.MakeDash(LDash, 0);
        ACanvas.DrawLine(10, 10, 100, 100, LPaint);
      finally
        ACanvas.Restore
      end;
    end;

Internally MakeDash calls the skia SkDashPathEffect method. The second argument to MakeDash is called the phase and details can be found at https://api.skia.org/classSkDashPathEffect.html

The code above will generate the following output:

[![enter image description here][1]][1]

  [1]: https://i.sstatic.net/UBR6w.png

**3. How to draw a bezier curve using skia?**

     procedure TfrmMain.SkPaintBox2Draw(ASender: TObject; const ACanvas: ISkCanvas;
       const ADest: TRectF; const AOpacity: Single);
     var LPaint :ISkPaint;
         PathBuilder : ISkPathBuilder;
        Path : ISkPath;
     begin
       ACanvas.Save;
       try
        LPaint := TSkPaint.Create (TSKPaintStyle.Stroke);
        LPaint.AntiAlias := True;
        LPaint.StrokeWidth := 2;
        LPaint.Color := claRed;

        PathBuilder := TSkPathBuilder.Create;
        PathBuilder.MoveTo(10, 10);
        PathBuilder.CubicTo(TPointF.Create(50, 20), TPointF.Create(150, 140), TPointF.Create(190, 90));

        ACanvas.DrawPath(PathBuilder.Detach, LPaint);
      finally
        ACanvas.Restore;
       end;
    end;

**4. How to draw circles**

      procedure TfrmMain.SkPaintBox3Draw(ASender: TObject; const ACanvas: ISkCanvas;
          const ADest: TRectF; const AOpacity: Single);
      var LPaint :ISkPaint;
      begin
        ACanvas.Save;
        try
          LPaint := TSkPaint.Create (TSKPaintStyle.Stroke);
          LPaint.AntiAlias := True;
      
          LPaint.StrokeWidth := 2;
          ACanvas.DrawCircle(TPointF.Create(50, 50), 20, LPaint);
          ACanvas.DrawCircle(TPointF.Create(150, 100), 25, LPaint);
      
          LPaint.Style := TSkPaintStyle.Fill;
      
          ACanvas.DrawCircle(TPointF.Create(50, 120), 15, LPaint);
      
          LPaint.Color := claOrange;
          ACanvas.DrawCircle(TPointF.Create(170, 170), 15, LPaint);

          // Draw two circles with fill and stroke at same position
          // using DrawCircle that takes the center point
          LPaint.Color := claOrange;
          LPaint.Style := TSkPaintStyle.Fill;
          ACanvas.DrawCircle(TPointF.Create(110, 160), 15, LPaint);
      
          LPaint.Color := claDarkOrange;
          LPaint.Style := TSkPaintStyle.Stroke;
          LPaint.StrokeWidth := 4;
          ACanvas.DrawCircle(TPointF.Create(110, 160), 15, LPaint);
        finally
          ACanvas.Restore
        end;
      end;

**5. How do I clear the canvas?**

      procedure TfrmMain.SkPaintBox4Draw(ASender: TObject; const ACanvas: ISkCanvas;
          const ADest: TRectF; const AOpacity: Single);
      var LPaint :ISkPaint;
      begin
        ACanvas.Save;
        try
          LPaint := TSkPaint.Create (TSKPaintStyle.Stroke);
      
          LPaint.Style :=  TSkPaintStyle.Fill;
          LPaint.Color := TAlphaColors.White;
          ACanvas.DrawRect(ADest, LPaint);
      
        finally
          ACanvas.Restore
        end;
      end;
      
**6. How to draw a rounded rectangle?**

      procedure TfrmMain.SkPaintBox12Draw(ASender: TObject; const ACanvas: ISkCanvas;
          const ADest: TRectF; const AOpacity: Single);
      var LPaint :ISkPaint;
      begin
        ACanvas.Save;
        try
          LPaint := TSkPaint.Create (TSKPaintStyle.Stroke);
          LPaint.AntiAlias := True;
      
          LPaint.StrokeWidth := 2;
          LPaint.Color := TAlphaColors.Darkred;
          ACanvas.DrawRoundRect(TRectF.Create(50, 40, 150, 140), 10, 10, LPaint);
        finally
          ACanvas.Restore
        end;
      end;

**7. How to draw some text**

    procedure TfrmMain.SkPaintBox13Draw(ASender: TObject; const ACanvas: ISkCanvas;
        const ADest: TRectF; const AOpacity: Single);
    var LPaint :ISkPaint;
        LFont : ISkFont;
        LTypeFace : ISKTypeFace;
        LBlob : ISkTextBlob;
    begin
      ACanvas.Save;
      try
        LPaint := TSkPaint.Create (TSKPaintStyle.Stroke);
        LPaint.AntiAlias := True;
    
        LTypeface := TSkTypeface.MakeFromName('Arial', TSkFontStyle.Normal);
        LFont := TSkFont.Create (LTypeface, 18, 1);
    
        LBlob := TSkTextBlob.MakeFromText('Some Text using Arial', LFont);
        LPaint.Style := TSKPaintStyle.Fill;
        LPaint.Color := claIndigo;
        ACanvas.DrawTextBlob(LBlob, 10, 40, LPaint);
    
        LPaint.Color := claDarkGreen;
        ACanvas.DrawSimpleText('Some simple text', 10, 100, LFont, LPaint);
      finally
        ACanvas.Restore
      end;
    end;

**8. How to draw a simple linear gradient?**

    procedure TfrmMain.SkPaintBox14Draw(ASender: TObject; const ACanvas: ISkCanvas;
        const ADest: TRectF; const AOpacity: Single);
    var LPaint :ISkPaint;
        LGradient : ISkShader;
    begin
      ACanvas.Save;
      try
        LPaint := TSkPaint.Create (TSKPaintStyle.Stroke);
        LPaint.AntiAlias := True;
    
        LPaint.Shader := TSkShader.MakeGradientLinear(TPointF.Create(0, 0), TPointF.Create(180, 10),
                        claBlue, claRed);
    
       ACanvas.DrawPaint(LPaint);
      finally
        ACanvas.Restore
      end;
    end;

**9. How to draw an oval**

    procedure TfrmMain.SkPaintBox15Draw(ASender: TObject; const ACanvas: ISkCanvas;
        const ADest: TRectF; const AOpacity: Single);
    var LPaint :ISkPaint;
    begin
      ACanvas.Save;
      try
        LPaint := TSkPaint.Create (TSKPaintStyle.Stroke);
        LPaint.AntiAlias := True;
        LPaint.StrokeWidth := 3;
        LPaint.Color := TAlphaColors.Blueviolet;
        ACanvas.DrawOval(TRectF.Create(20,50, 180, 90), LPaint);
      finally
        ACanvas.Restore
      end;
    end;

**10. How to draw arcs**

procedure TfrmMain.SkPaintBox16Draw(ASender: TObject; const ACanvas: ISkCanvas;
    const ADest: TRectF; const AOpacity: Single);
var LPaint :ISkPaint;
begin
  ACanvas.Save;
  try
    LPaint := TSkPaint.Create (TSKPaintStyle.Stroke);
    LPaint.AntiAlias := True;

    LPaint.StrokeWidth := 3;
    LPaint.Color := TAlphaColors.Blue;
    // Open arc
    ACanvas.DrawArc(TRectF.Create(40, 60, 120, 140), 0, 255, False, LPaint);

    LPaint.Color := TAlphaColors.Red;
    // Closed arc
    ACanvas.DrawArc(TRectF.Create(20, 20, 170, 170), 0, 255, True, LPaint);
  finally
    ACanvas.Restore
  end;
end;

**11. How to load an image**

      procedure TfrmMain.SkPaintBox17Draw(ASender: TObject; const ACanvas: ISkCanvas;
          const ADest: TRectF; const AOpacity: Single);
      var LPaint :ISkPaint;
          Image : ISKImage;
      begin
        ACanvas.Save;
        try
          LPaint := TSkPaint.Create (TSKPaintStyle.Stroke);
          LPaint.AntiAlias := True;
      
          Image := TSKImage.MakeFromEncodedFile('image.png');
          ACanvas.DrawImage(Image, 0, 0, LPaint);
        finally
          ACanvas.Restore
        end;
      end;

**12. How to create a fill of lines hatch horizontal lines**

      procedure TfrmMain.SkPaintBox18Draw(ASender: TObject; const ACanvas: ISkCanvas;
          const ADest: TRectF; const AOpacity: Single);
      var LPaintFill, LPaintStroke :ISkPaint;
          PathBuilder : ISkPathBuilder;
          Path : ISkPath;
          PathEffect : ISkPathEffect;
          m : TMatrix;  // Comes from Math.Vectors
      begin
        ACanvas.Save;
        try
          LPaintFill := TSkPaint.Create (TSKPaintStyle.Fill);
          LPaintFill.AntiAlias := True;
      
          LPaintStroke := TSkPaint.Create (TSKPaintStyle.Stroke);
          LPaintStroke.AntiAlias := True;

          // Create a diamond
          PathBuilder := TSkPathBuilder.Create;
          PathBuilder.MoveTo(0, 150 / 2);
          PathBuilder.LineTo(150 / 2, 0);
          PathBuilder.LineTo(150, 150 / 2);
          PathBuilder.LineTo(150 / 2, 150);
          PathBuilder.Close();
          Path := PathBuilder.Detach;
      
          LPaintFill.StrokeWidth := 1;
          // Set the distance between the lines
          m := TMatrix.CreateScaling (1, 20);
          // Create parallel lines
          PathEffect := TSkPathEffect.Make2DLine(1, m);
          LPaintFill.PathEffect := PathEffect;
          ACanvas.DrawPath(Path, LPaintFill);
      
          // Put a border aroudn the hatch
          LPaintStroke.StrokeWidth := 1;
          ACanvas.DrawPath (Path, LPaintStroke);
        finally
          ACanvas.Restore
        end;
      end;

   **12 How to save a canvas drawing to a pdf file**

      procedure TRRGraph.exportToPDF (filename : string);
      var LPDFStream: TStream;
          LDocument: ISkDocument;
          LCanvas : ISkCanvas;
          i : integer;
      begin
        LPDFStream := TFileStream.Create(fileName, fmCreate);
        try
          LDocument := TSkDocument.MakePDF(LPDFStream);
          try
            LCanvas := LDocument.BeginPage(Width, Height);
            try
             LCanvas.Clear(TAlphaColors.White);
             drawToCanvas(LCanvas);
            finally
              LDocument.EndPage;
            end;
          finally
            LDocument.Close;
          end;
        finally
          LPDFStream.Free;
        end;
      end;
