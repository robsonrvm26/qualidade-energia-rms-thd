# qualidade-energia-rms-thd
Análise e Classificação de Distúrbios de Qualidade de Energia via RMS e THD utilizando MATLAB.
qualidade-energia-rms-thd
│
├── Codigo_MATLAB
│      classificacao_disturbios.m
│add file
upload file
clc;
clear;
close all;

%% Parâmetros gerais
fs = 10000;              % Frequência de amostragem (Hz)
f = 60;                  % Frequência fundamental (Hz)
t = 0:1/fs:0.2;          % Tempo de simulação (s)
Vm = 1;                  % Amplitude nominal (pu)

%% Sinal normal
v_normal = Vm*sin(2*pi*f*t);

%% Sag - afundamento de tensão
v_sag = v_normal;
idx_sag = (t >= 0.06 & t <= 0.14);
v_sag(idx_sag) = 0.5*sin(2*pi*f*t(idx_sag));

%% Swell - sobretensão temporária
v_swell = v_normal;
idx_swell = (t >= 0.06 & t <= 0.14);
v_swell(idx_swell) = 1.3*sin(2*pi*f*t(idx_swell));

%% Harmônicas
v_harmonico = sin(2*pi*f*t) + ...
              0.30*sin(2*pi*3*f*t) + ...
              0.20*sin(2*pi*5*f*t);

%% Flicker - modulação lenta da amplitude
modulacao = 1 + 0.15*sin(2*pi*10*t);
v_flicker = modulacao .* sin(2*pi*f*t);

%% Transiente
v_transiente = v_normal;
idx_tr = (t >= 0.08 & t <= 0.085);
v_transiente(idx_tr) = v_transiente(idx_tr) + ...
                       2*exp(-800*(t(idx_tr)-0.08));

%% Desequilíbrio trifásico
Va = sin(2*pi*f*t);
Vb = 0.85*sin(2*pi*f*t - 2*pi/3);
Vc = 1.10*sin(2*pi*f*t + 2*pi/3);

%% Múltiplos distúrbios simultâneos
v_multi = v_normal;
idx_multi = (t >= 0.06 & t <= 0.14);
v_multi(idx_multi) = 0.6*sin(2*pi*f*t(idx_multi));
v_multi = v_multi + 0.25*sin(2*pi*3*f*t) + ...
                    0.15*sin(2*pi*5*f*t);
v_multi(idx_tr) = v_multi(idx_tr) + ...
                  1.5*exp(-900*(t(idx_tr)-0.08));

%% Funções de cálculo
calc_rms = @(x) sqrt(mean(x.^2));
calc_thd = @(x) calcularTHD(x, fs, f);
taxa_variacao = @(x) max(abs(diff(x))) * fs;

%% Cálculo dos parâmetros
sinais = {v_normal, v_sag, v_swell, v_harmonico, v_flicker, v_transiente, v_multi};
nomes = ["Normal","Sag","Swell","Harmônicas","Flicker","Transiente","Múltiplo"];

RMS = zeros(1,length(sinais));
THD = zeros(1,length(sinais));
TVT = zeros(1,length(sinais));
classe = strings(1,length(sinais));

for i = 1:length(sinais)
    RMS(i) = calc_rms(sinais{i});
    THD(i) = calc_thd(sinais{i});
    TVT(i) = taxa_variacao(sinais{i});
    classe(i) = classificarSinal(RMS(i), THD(i), TVT(i), sinais{i});
end

%% Exibir resultados no Command Window
fprintf('\n--- RESULTADOS DOS SINAIS ---\n');

for i = 1:length(sinais)
    fprintf('%s | RMS = %.4f pu | THD = %.2f %% | TVT = %.2f | Classe = %s\n', ...
        nomes(i), RMS(i), THD(i)*100, TVT(i), classe(i));
end

%% Figura 1 - Sinal normal
figure;
plot(t, v_normal, 'LineWidth', 1.2);
grid on;
title('Sinal Senoidal Normal');
xlabel('Tempo (s)');
ylabel('Amplitude (pu)');

%% Figura 2 - Sag
figure;
plot(t, v_sag, 'LineWidth', 1.2);
grid on;
title('Distúrbio: Afundamento de Tensão (Sag)');
xlabel('Tempo (s)');
ylabel('Amplitude (pu)');

%% Figura 3 - Swell
figure;
plot(t, v_swell, 'LineWidth', 1.2);
grid on;
title('Distúrbio: Sobretensão Temporária (Swell)');
xlabel('Tempo (s)');
ylabel('Amplitude (pu)');

%% Figura 4 - Harmônicas
figure;
plot(t, v_harmonico, 'LineWidth', 1.2);
grid on;
title('Distúrbio: Distorção Harmônica');
xlabel('Tempo (s)');
ylabel('Amplitude (pu)');

%% Figura 5 - Flicker
figure;
plot(t, v_flicker, 'LineWidth', 1.2);
grid on;
title('Distúrbio: Flicker');
xlabel('Tempo (s)');
ylabel('Amplitude (pu)');

%% Figura 6 - Transiente
figure;
plot(t, v_transiente, 'LineWidth', 1.2);
grid on;
title('Distúrbio: Transiente Elétrico');
xlabel('Tempo (s)');
ylabel('Amplitude (pu)');

%% Figura 7 - Múltiplos distúrbios simultâneos
figure;
plot(t, v_multi, 'LineWidth', 1.2);
grid on;
title('Múltiplos Distúrbios Simultâneos');
xlabel('Tempo (s)');
ylabel('Amplitude (pu)');

%% Figura 8 - Comparação RMS
figure;
bar(RMS);
grid on;
title('Comparação dos Valores RMS');
ylabel('RMS (pu)');
set(gca, 'XTickLabel', nomes);

%% Figura 9 - Comparação THD
figure;
bar(THD*100);
grid on;
title('Comparação da Distorção Harmônica Total');
ylabel('THD (%)');
set(gca, 'XTickLabel', nomes);

%% Figura 10 - Taxa de variação de tensão
figure;
bar(TVT);
grid on;
title('Comparação da Taxa de Variação de Tensão');
ylabel('TVT');
set(gca, 'XTickLabel', nomes);

%% Figura 11 - FFT do sinal harmônico
N = length(v_harmonico);
Y = fft(v_harmonico);
P2 = abs(Y/N);
P1 = P2(1:floor(N/2)+1);
P1(2:end-1) = 2*P1(2:end-1);
f_fft = fs*(0:floor(N/2))/N;

figure;
plot(f_fft, P1, 'LineWidth', 1.2);
grid on;
title('FFT do Sinal com Harmônicas');
xlabel('Frequência (Hz)');
ylabel('Amplitude');
xlim([0 500]);

%% Figura 12 - Desequilíbrio trifásico
figure;
plot(t, Va, 'LineWidth', 1.2);
hold on;
plot(t, Vb, 'LineWidth', 1.2);
plot(t, Vc, 'LineWidth', 1.2);
grid on;
title('Desequilíbrio de Tensão Trifásica');
xlabel('Tempo (s)');
ylabel('Amplitude (pu)');
legend('Va','Vb','Vc');

%% Figura 13 - Matriz de confusão
real = ["Normal","Sag","Swell","Harmônicas","Flicker","Transiente","Múltiplo"];
previsto = classe;

figure;
confusionchart(real, previsto);
title('Matriz de Confusão da Classificação dos Distúrbios');

accuracy = sum(real == previsto)/length(real);
fprintf('\nAcurácia = %.2f %%\n', accuracy*100);

%% Função THD
function thd = calcularTHD(signal, fs, f)

    N = length(signal);
    Y = fft(signal);
    Y = abs(Y/N);
    f_axis = (0:N-1)*(fs/N);

    [~, idx1] = min(abs(f_axis - f));
    V1 = Y(idx1);

    Vh = 0;

    for k = 2:10
        [~, idx] = min(abs(f_axis - k*f));
        Vh = Vh + Y(idx)^2;
    end

    thd = sqrt(Vh)/V1;

end

%% Função de classificação
function classe = classificarSinal(rms_val, thd_val, tvt_val, sinal)

    rms_nominal = 0.707;

    if thd_val > 0.05 && rms_val < 0.9*rms_nominal
        classe = "Múltiplo";
    elseif rms_val < 0.9*rms_nominal
        classe = "Sag";
    elseif rms_val > 1.1*rms_nominal
        classe = "Swell";
    elseif thd_val > 0.05
        classe = "Harmônicas";
    elseif tvt_val > 60000
        classe = "Transiente";
    elseif std(abs(sinal)) > 0.35 && thd_val < 0.05
        classe = "Flicker";
    else
        classe = "Normal";
    end

endloading analise_e_comparacao_da_qualidade_de_energia_graficos_tabelas.m…]()

├── Graficos
│      Sag.png
│      Swell.png
│      Harmonica.png
│      FFT.png
│      Comparacao_RMS.png
│      Comparacao_TVT.png
│      Matriz_Confusao.png
│
├── Imagens_Experimentais
│      Ambiente_Industrial.jpg
│      Osciloscopio.jpg
│      Banco_Capacitores.jpg
│
├── Artigo
│      TCC_Qualidade_Energia.pdf
│
└── README.md
