%% Taller 2 - Respuesta de un sistema a señales arbitrarias
% Archivo: taller2_actividadreto.m
%Johanna Gabriela Coronel Criollo
clear; close all; clc;

%% Parámetros del sistema
% Función de transferencia:
%          3
% G(s)= ---------
%       s^2+2s+3
num = [3];
den = [1 2 3];
G = tf(num, den); % sistema continuo sin retardo

% Definir el sistema con retardo (InputDelay = 2 s) para las actividades que lo requieran
delay_sec = 2;
G_delay = tf(num, den, 'InputDelay', delay_sec);

%% 1) Respuesta a señales típicas: impulso y escalón (sin delay)
tfinal = 10;                    % tiempo final para simulaciones
t = linspace(0, tfinal, 1000);  % vector de tiempo

% --- Impulso ---
[yi, ti] = impulse(G, t);
figure('Name','Respuesta al Impulso (sin delay)');
plot(ti, yi, 'LineWidth', 1.5);
grid on; xlabel('Tiempo [s]'); ylabel('Salida'); title('Respuesta al Impulso de G(s)');
% marcar máximo
[yi_max, idxi_max] = max(yi);
ti_max = ti(idxi_max);
hold on; plot(ti_max, yi_max, 'ro', 'MarkerSize',8, 'DisplayName','Máximo');
legend('Respuesta impulso','Máximo','Location','best');

% --- Escalón ---
[ys, ts] = step(G, t);
figure('Name','Respuesta al Escalón (sin delay)');
plot(ts, ys, 'LineWidth', 1.5);
grid on; xlabel('Tiempo [s]'); ylabel('Salida'); title('Respuesta al Escalón de G(s)');
% marcar máximo
[ys_max, idxs_max] = max(ys);
ts_max = ts(idxs_max);
hold on; plot(ts_max, ys_max, 'ro', 'MarkerSize',8, 'DisplayName','Máximo');
legend('Respuesta escalón','Máximo','Location','best');

%% 2) Sistema con retardo: graficar impulse y step (con delay)
% Para visualizar el efecto del retardo usamos G_delay
t_delay = linspace(0, tfinal+delay_sec, 1200);

[yid, tid] = impulse(G_delay, t_delay);
figure('Name','Impulso (con retardo)');
plot(tid, yid, 'LineWidth', 1.5);
grid on; xlabel('Tiempo [s]'); ylabel('Salida'); title(sprintf('Respuesta al Impulso de G(s) con retardo de %g s', delay_sec));
% marcar máximo
[yid_max, idyid_max] = max(yid);
tid_max = tid(idyid_max);
hold on; plot(tid_max, yid_max, 'ro', 'MarkerSize',8);

[ysd, tsd] = step(G_delay, t_delay);
figure('Name','Escalón (con retardo)');
plot(tsd, ysd, 'LineWidth', 1.5);
grid on; xlabel('Tiempo [s]'); ylabel('Salida'); title(sprintf('Respuesta al Escalón de G(s) con retardo de %g s', delay_sec));
% marcar máximo
[ysd_max, idysd_max] = max(ysd);
tsd_max = tsd(idysd_max);
hold on; plot(tsd_max, ysd_max, 'ro', 'MarkerSize',8);

%% 3) Analizar tiempo en que el sistema empieza a responder (por retardo)
umbral = 1e-3; % umbral para considerar "inicio de respuesta"
idx_inicio_imp = find(abs(yid) > umbral, 1, 'first');
t_inicio_imp = tid(idx_inicio_imp); % debería ser aproximadamente delay_sec
idx_inicio_step = find(abs(ysd - ysd(1)) > umbral, 1, 'first');
t_inicio_step = tsd(idx_inicio_step);

fprintf('Tiempo estimado de inicio de respuesta (impulso con retardo): %.4f s\n', t_inicio_imp);
fprintf('Tiempo estimado de inicio de respuesta (escalón con retardo): %.4f s\n', t_inicio_step);

%% 4) Respuesta a señales arbitrarias (Figura 2.2 y 2.3)
% Ejemplo señal arbitraria 2.2 (crearemos una señal compuesta: escalón breve, senoidal y pulso)
t_arbitrary = linspace(0, tfinal+delay_sec, 1200);
signal1 = zeros(size(t_arbitrary));
% crear una pequeña secuencia de cambios tipo figura 2.2(a)
signal1(t_arbitrary>=0 & t_arbitrary<1) = 0;                 % 0 a 1s
signal1(t_arbitrary>=1 & t_arbitrary<2.5) = 1;               % escalón a 1 desde 1s a 2.5s
signal1 = signal1 + 0.5*sin(2*pi*0.7*t_arbitrary).*exp(-0.1*t_arbitrary); % añadir seno amortiguado
signal1(t_arbitrary>=5 & t_arbitrary<5.5) = signal1(t_arbitrary>=5 & t_arbitrary<5.5) + 1.2; % pulso añadido

figure('Name','Señal Arbitraria (Figura 2.2 - ejemplo)');
plot(t_arbitrary, signal1, 'LineWidth',1.2);
grid on; xlabel('Tiempo [s]'); ylabel('Amplitud'); title('Señal Arbitraria - Ejemplo (Fig.2.2.a)');

% Simular respuesta con retardo:
y_arbitrary1 = lsim(G_delay, signal1, t_arbitrary);
figure('Name','Respuesta a señal arbitraria (Fig 2.2) - con retardo');
plot(t_arbitrary, y_arbitrary1, 'LineWidth',1.2);
grid on; xlabel('Tiempo [s]'); ylabel('Salida'); title('Respuesta de G(s) con retardo a señal arbitraria (Fig2.2)');
% marcar máximo en la respuesta
[yar1_max, idx_yar1_max] = max(y_arbitrary1);
t_yar1_max = t_arbitrary(idx_yar1_max);
hold on; plot(t_yar1_max, yar1_max, 'ro', 'MarkerSize',8);

%% 5) Repetir para la señal de la Figura 2.3 
signal2 = zeros(size(t_arbitrary));
% construimos: rampa ascendente, escalón, rampa descendente, escalón de bajada
signal2 = signal2 + (t_arbitrary>=0 & t_arbitrary<2).* ( (t_arbitrary-0)/2 );    % rampa asc a 2s (0->1)
signal2(t_arbitrary>=2 & t_arbitrary<3.5) = 1.5;                                % escalón a 1.5
signal2 = signal2 + (t_arbitrary>=3.5 & t_arbitrary<5.5).* ( - (t_arbitrary-3.5)/2 ); % rampa descendente
signal2(t_arbitrary>=5.5) = 0.2; % escalón de bajada

figure('Name','Señal Arbitraria (Figura 2.3 - ejemplo)');
plot(t_arbitrary, signal2, 'LineWidth',1.2);
grid on; xlabel('Tiempo [s]'); ylabel('Amplitud'); title('Señal Arbitraria - Ejemplo (Fig.2.3)');

y_arbitrary2 = lsim(G_delay, signal2, t_arbitrary);
figure('Name','Respuesta a señal arbitraria (Fig 2.3) - con retardo');
plot(t_arbitrary, y_arbitrary2, 'LineWidth',1.2);
grid on; xlabel('Tiempo [s]'); ylabel('Salida'); title('Respuesta de G(s) con retardo a señal arbitraria (Fig2.3)');
% marcar máximo
[yar2_max, idx_yar2_max] = max(y_arbitrary2);
t_yar2_max = t_arbitrary(idx_yar2_max);
hold on; plot(t_yar2_max, yar2_max, 'ro', 'MarkerSize',8);

%% ----------------------------
%% Actividad Reto
% Generar una señal aleatoria que incluya: una rampa ascendente, un escalón de subida,
% una rampa descendente y un escalón de bajada. Además sumamos ruido aleatorio.
t_reto = linspace(0, 12, 1200)'; % tiempo para actividad reto (0 a 12 s)
sr = 1/(t_reto(2)-t_reto(1));     % frecuencia de muestreo efectiva

% Construcción piecewise:
s_reto = zeros(size(t_reto));
% 0-2s: rampa ascendente (0->1)
idx = find(t_reto>=0 & t_reto<2);
s_reto(idx) = (t_reto(idx)-0)/2;
% 2-4s: escalón de subida a 1.8
s_reto(t_reto>=2 & t_reto<4) = 1.8;
% 4-7s: rampa descendente a 0.2
idx2 = find(t_reto>=4 & t_reto<7);
s_reto(idx2) = 1.8 - ( (t_reto(idx2)-4) * (1.6/3) ); % baja linealmente de 1.8 a 0.2 en 3s
% 7-9s: escalón de bajada a 0.2 (ya lo dejó)
s_reto(t_reto>=7 & t_reto<9) = 0.2;
% 9-12s: ruido aleatorio + leve oscilación
s_reto(t_reto>=9) = 0.2 + 0.15*randn(sum(t_reto>=9),1) + 0.1*sin(2*pi*0.5*t_reto(t_reto>=9));

% Añadir una componente aleatoria leve sobre todo el rango (para "aleatoriedad")
rng(0); % para reproducibilidad
s_reto = s_reto + 0.05*randn(size(s_reto));

% Grafica la señal reto
figure('Name','Actividad Reto - Señal Aleatoria compuesta');
plot(t_reto, s_reto, 'LineWidth',1.2);
grid on; xlabel('Tiempo [s]'); ylabel('Amplitud'); title('Actividad Reto: Señal aleatoria con rampas y escalones');
xlim([0 max(t_reto)]);

% Simular respuesta con el sistema G_delay
y_reto = lsim(G_delay, s_reto, t_reto);

figure('Name','Actividad Reto - Respuesta del Sistema');
plot(t_reto, y_reto, 'LineWidth',1.2);
hold on; plot(t_reto, s_reto, '--', 'LineWidth',0.8); % comparar entrada (dashed)
grid on; xlabel('Tiempo [s]'); ylabel('Amplitud'); legend('Salida', 'Entrada (referencia)');
title('Actividad Reto: Respuesta de G(s) con retardo a la señal aleatoria');

% Marcar máximos y calcular métricas
[y_reto_max, idx_yreto_max] = max(y_reto);
t_yreto_max = t_reto(idx_yreto_max);
plot(t_yreto_max, y_reto_max, 'ro', 'MarkerSize',8, 'DisplayName','Máximo salida');

% Calcular error RMS entre entrada y salida 
err = y_reto - s_reto;
RMS_error = sqrt(mean(err.^2));
fprintf('RMS error (entrada vs salida) para Actividad Reto: %.6f\n', RMS_error);

% Encontrar tiempo de inicio de respuesta (umbral)
umbral_reto = 1e-3;
idx_inicio = find(abs(y_reto) > umbral_reto, 1, 'first');
t_inicio_reto = t_reto(idx_inicio);
fprintf('Tiempo estimado de inicio de respuesta (Actividad Reto): %.4f s\n', t_inicio_reto);

%% Guardar resultados relevantes en workspace para reporte
resultado.t_reto = t_reto;
resultado.s_reto = s_reto;
resultado.y_reto = y_reto;
resultado.RMS_error = RMS_error;
resultado.t_inicio_reto = t_inicio_reto;
resultado.y_reto_max = y_reto_max;
resultado.t_yreto_max = t_yreto_max;

save('resultados_taller2_actividadreto.mat', 'resultado');
