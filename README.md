# FMX-Skia-Tips
A list of tips for users of skia and FmX

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
