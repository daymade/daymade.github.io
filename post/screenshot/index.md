
<!--more-->

```delphi
unit1
        
	unit Unit1;

        interface

        uses
          Windows, Messages, SysUtils, Variants, Classes, Graphics, Controls, Forms,
          Dialogs, ExtCtrls, ExtDlgs, StdCtrls,Jpeg;

        type
          TForm1 = class(TForm)
            Button1: TButton;
            Button2: TButton;
            Button3: TButton;
            Button4: TButton;
            Image1: TImage;
            Timer1: TTimer;
            SavePictureDialog1: TSavePictureDialog;
            Label1: TLabel;
            procedure Button3Click(Sender: TObject);
            procedure Button4Click(Sender: TObject);
            procedure Button1Click(Sender: TObject);
            procedure Timer1Timer(Sender: TObject);
            procedure Button2Click(Sender: TObject);
            procedure Label1Click(Sender: TObject);
          private
            { Private declarations }
          public
            { Public declarations }
          end;

        var
          Form1: TForm1;
        implementation

        uses Unit2, Unit4;

        {$R *.dfm}

        procedure TForm1.Button1Click(Sender: TObject);
        begin
            form1.Hide;
            timer1.Enabled:=true;
        end;

        procedure TForm1.Button2Click(Sender: TObject);
        begin
          form1.Hide;
          form2.show;
        end;

        procedure SaveAsJPG ;
        var
          jpg: TJPEGImage;
        begin
          jpg := TJPEGImage.Create;
          try
            with jpg do
            begin
              CompressionQuality := 42;
              Compress ;
              Assign(form1.Image1.Picture.Bitmap);
              SaveToFile(form1.savepicturedialog1.FileName+'.jpg');
            end;
          finally
            jpg.Free;
          end;
          messagebeep(mb_ok);
        end;

        procedure SaveAsBMP ;
        begin
          with form1.image1.Picture do
            SaveToFile(form1.savepicturedialog1.FileName+'.bmp');
          messagebeep(mb_ok);
        end;

        procedure TForm1.Button3Click(Sender: TObject);
        begin
             if SavePictureDialog1.Execute then
               begin
                 if savepicturedialog1.filterindex = 1 then
                   SaveAsJPG
                else
                   SaveAsBMP;
               end
        end;

        procedure TForm1.Button4Click(Sender: TObject);
        begin
         form1.close;
        end;

        procedure TForm1.Label1Click(Sender: TObject);
        begin
          form4.show;
        end;

        procedure TForm1.Timer1Timer(Sender: TObject);
        var
            Fullscreen:Tbitmap;
            Fullscreencanvas:tcanvas;
            dc:HDC;
        begin
              timer1.Enabled:=false;
              fullscreen:=tbitmap.create;
              fullscreen.width:=screen.Width;
              fullscreen.Height:=screen.Height;
              dc:=getdc(0);
              fullscreencanvas:=tcanvas.Create;
              fullscreencanvas.Handle:=dc;
              fullscreen.Canvas.CopyRect(rect(0,0,screen.Width,screen.Height),
              fullscreencanvas,rect(0,0,screen.Width,screen.Height));
              fullscreencanvas.Free;
              releasedc(0,dc);
              image1.Picture.Bitmap:=fullscreen;
              fullscreen.Free;
              form1.Show;
              messagebeep(mb_ok);
        end;

        end.
        {//reference
        Choose SaveAsJPG and SaveAsBMP using savepicturedialog
          http://www.delphigroups.info/2/38/204152.html
        BMP2JPG
          http://docwiki.embarcadero.com/CodeExamples/XE4/en/TJPEGImageAssign_(Delphi)
        AreaSnap function
          http://blog.sina.com.cn/s/blog_3ec38a670100064s.html
        Icon
          https://www.iconfinder.com/icons/64677/snapshot_icon#size=32
        }
unit2

        unit Unit2;

        interface

        uses
          Windows, Messages, SysUtils, Variants, Classes, Graphics, Controls, Forms,
          Dialogs, StdCtrls;

        type
          TForm2 = class(TForm)
            Label1: TLabel;
            Label2: TLabel;
            Button1: TButton;
            procedure FormClose(Sender: TObject; var Action: TCloseAction);
            procedure Button1Click(Sender: TObject);
          private
            { Private declarations }
            procedure   CreateParams(var Params:TCreateParams);   override;
          public
            { Public declarations }
          end;

        var
          Form2: TForm2;

        implementation

        uses Unit1, Unit3;

        {$R *.dfm}
        procedure TForm2.Button1Click(Sender: TObject);
        begin
          form2.Hide;
          form3.timer1.Enabled:=true;
        end;

        procedure   TForm2.CreateParams(var Params:TCreateParams);
        begin
            inherited;
            Params.WndParent:=   0;
        end;

        procedure TForm2.FormClose(Sender: TObject; var Action: TCloseAction);
        begin
           form2.Hide;
           sleep(300);
           form1.Show;
        end;

        end.
unit3

        unit Unit3;

        interface

        uses
          Windows, Messages, SysUtils, Variants, Classes, Graphics, Controls, Forms,
          Dialogs, ExtCtrls, StdCtrls;

        type
          TForm3 = class(TForm)
            Timer1: TTimer;
            Image1: TImage;
            Label1: TLabel;
            Timer2: TTimer;
            procedure Timer1Timer(Sender: TObject);
            procedure FormKeyDown(Sender: TObject; var Key: Word; Shift: TShiftState);
            procedure FormClose(Sender: TObject; var Action: TCloseAction);
            procedure Timer2Timer(Sender: TObject);
            procedure FormCreate(Sender: TObject);
            procedure Image1MouseDown(Sender: TObject; Button: TMouseButton;
              Shift: TShiftState; X, Y: Integer);
            procedure Image1MouseMove(Sender: TObject; Shift: TShiftState; X,
              Y: Integer);
          private
            { Private declarations }
          public
            { Public declarations }
          end;

        var
          Form3: TForm3;
              foldx,x1,y1,x2,y2,oldx,oldy,foldy:integer;
            flag,trace:boolean;
        implementation

        uses Unit1, Unit2;

        {$R *.dfm}

        procedure TForm3.FormClose(Sender: TObject; var Action: TCloseAction);
        begin
          form3.free;
          form1.show;
        end;

        procedure TForm3.FormCreate(Sender: TObject);
        begin
          timer2.Enabled:= true;
        end;

        procedure TForm3.FormKeyDown(Sender: TObject; var Key: Word;
          Shift: TShiftState);
        begin
         if key=27 then
           form3.Close();
        end;

        procedure TForm3.Image1MouseDown(Sender: TObject; Button: TMouseButton;
          Shift: TShiftState; X, Y: Integer);
        var
            width,height:integer;
            newbitmap:TBitmap;
        begin
            if(trace=false) then
              begin
                 flag:=false;
                 with image1.Canvas do
                 begin
                     moveto(foldx,0);
                     lineto(foldx,screen.Height);
                     moveto(0,foldy);
                     lineto(screen.Width,foldy);
                 end;
                 x1:=x;
                 y1:=y;
                 oldx:=x;
                 oldy:=y;
                 trace:=true;
                 image1.Canvas.Pen.Color:=clblack;
                 image1.Canvas.Brush.Style:=bsclear;
              end
            else
              begin
                 x2:=x;
                 y2:=y;
                 trace:=false;
                 image1.Canvas.Rectangle(x1,y1,oldx,oldy);
                 width:=abs(x2-x1);
                 height:=abs(y2-y1);
                 form3.Image1.Width:=width;
                 form3.Image1.Height:=height;
                 newbitmap:=Tbitmap.Create;
                 newbitmap.Width:=width;
                 newbitmap.Height:=height;
                 newbitmap.Canvas.CopyRect(rect(0,0,width,height),form3.Image1.Canvas,
                                      rect(x1,y1,x2,y2));
                form1.Image1.Picture.Bitmap:=newbitmap;
                newbitmap.Free;
                form3.Hide;
                sleep(300);
                form1.Show;
            end;

        end;

        procedure TForm3.Image1MouseMove(Sender: TObject; Shift: TShiftState; X,
          Y: Integer);
        begin
            if trace=true then
              begin
                with image1.Canvas do
                begin
                    rectangle(x1,y1,oldx,oldy);
                    rectangle(x1,y1,x,y);
                    oldx:=x;
                    oldy:=y;
                end;
            end
            else  if flag=true then
            begin
                with image1.Canvas do
                begin
                    moveto(foldx,0);
                    lineto(foldx,screen.Height);
                    moveto(0,foldy);
                    lineto(screen.Width,foldy);
                    moveto(x,0);
                    lineto(x,screen.Height);
                    moveto(0,y);
                    lineto(screen.Width,y);
                    foldx:=x;
                    foldy:=y;
               end;
            end;

        end;

        procedure TForm3.Timer1Timer(Sender: TObject);

        var
            Fullscreen:Tbitmap;
            Fullscreencanvas:tcanvas;
            dc:HDC;
        begin
              timer1.Enabled:=false;
              fullscreen:=tbitmap.create;
              fullscreen.width:=screen.Width;
              fullscreen.Height:=screen.Height;
              dc:=getdc(0);
              fullscreencanvas:=tcanvas.Create;
              fullscreencanvas.Handle:=dc;
              fullscreen.Canvas.CopyRect(rect(0,0,screen.Width,screen.Height),
              fullscreencanvas,rect(0,0,screen.Width,screen.Height));
              fullscreencanvas.Free;
              releasedc(0,dc);
              form3.image1.Picture.Bitmap:=fullscreen;
              fullscreen.Free;
              form3.Show;
              messagebeep(mb_ok);
              foldx:=-1;
              foldy:=-1;
              image1.Canvas.Pen.Mode:=pmnot;
              image1.Canvas.Pen.Color:=clblack;
              image1.Canvas.Brush.Style:=bsclear;
              flag:=true;
        end;

        procedure TForm3.Timer2Timer(Sender: TObject);
        begin
        label1.Visible := Not label1.visible;
        end;


        end.
unit4

        unit Unit4;

        interface

        uses
          Windows, Messages, SysUtils, Variants, Classes, Graphics, Controls, Forms,
          Dialogs, StdCtrls;

        type
          TForm4 = class(TForm)
            Button1: TButton;
            Label1: TLabel;
            procedure Button1Click(Sender: TObject);
            procedure FormDeactivate(Sender: TObject);
          private
            { Private declarations }
          public
            { Public declarations }
          end;

        var
          Form4: TForm4;

        implementation

        {$R *.dfm}

        procedure TForm4.Button1Click(Sender: TObject);
        begin
          form4.hide;
        end;

        procedure TForm4.FormDeactivate(Sender: TObject);
        begin
          form4.hide;
        end;

        end.
```