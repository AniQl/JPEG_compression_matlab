clc
clear
a = double(imread('lena.png'));
sizeofimage=size(a);
dlmwrite('lena.txt',a,'delimiter',' ');

file = textread('lena.txt', '%u' );
file=file';
k=1;
for i=1:sizeofimage
    for j=1:sizeofimage
        
        M(i,j)=file(k);
        k=k+1;
    end
end
prompt = 'What blocksize u want to use, define x: ';
prompt2 = 'What blocksize u want to use, define y: ';
x = input(prompt)
y = input(prompt2)
blocksize=[x y];
M=uint8(M);
subplot(1,3,1)
imshow(M);

%dct

M = im2double(M);
T = dctmtx(x);
dct2 = @(block_struct) T * block_struct.data * T';
B = blockproc(M,blocksize,dct2);
dlmwrite('lenadct.txt',B,'delimiter',' ');
subplot(1,3,2)
imshow(B)

%idct

invdct = @(block_struct) T' * block_struct.data * T;
C = blockproc(B,blocksize, invdct);
dlmwrite('lenaidct.txt',C,'delimiter',' ');
subplot(1,3,3)
imshow(C)
M= uint8(M);
C=uint8(C);

%quantization?

Quantization_Table = [16 11 10 16 24 40 51 61; 12 12 14 19 26 58 60 55; 14 13 16 24 40 57 69 56; 14 17 22 29 51 87 80 62; 18 22 37 56 68 109 103 77; 24 35 55 64 81 104 113 92; 49 64 78 87 103 121 120 101; 72 92 95 98 112 100 103 99];
QA = @(block_struct)(block_struct.data) ./ Quantization_Table;
B2 = blockproc(B,[x y],QA);
%B2 = round(B2);
B3 = blockproc(B2,[x y],@(block_struct) Quantization_Table .* block_struct.data);
invdct = @(block_struct) round(T' * block_struct.data * T);
I2 = blockproc(B3,[x y],invdct);
figure;
imshow(I2);

%zigzag
zigzag = reshape(1:numel(I2), size(I2));
zigzag = fliplr( spdiags( fliplr(zigzag) ) );
zigzag(:,1:2:end) = flipud( zigzag(:,1:2:end) );
zigzag(zigzag==0) = [];
D=I2(zigzag);

%RLE scanning
F=[logical(diff(D)) 1];
In=find(F~=0);
Ele=D(In);
C=[In(1) diff(In)];

Result=zeros([1 numel(Ele) ]);
Result(1,:)=Ele;
Result(2,:)=C;

n=min(length(Ele),length(C));
output=[];

for t=1:n
    output=[output Ele(t) C(t)];
end

%prediction