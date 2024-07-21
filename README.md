# FMX-Skia-Tips

A list of tips for users of skia and FMX

1. How to draw a line or bezier curve witha rounded end?
2. How to create a dotted line?
3. How to draw a bezier curve using skia?
4. How to draw circles
5. How do I clear the canvas?
6. How to draw a rounded rectangle?
7. How to draw some Text?
  

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
        ACanvas.DrawTextBlob(LBlob, 10, 30, LPaint);
      finally
        ACanvas.Restore
      end;
    end;
