# FMX-Skia-Tips
A list of tips for users of skia and FMX

**1. How do I draw a line or bezier curve witha rounded end?**
   
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

**2. How do I create a dotted line using skia4Delphi?**

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

**3. How do I draw a bezier curve using skia?**

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
    finally
      ACanvas.Restore
    end;
  end;

