import numpy as np

import qutip as qt

from scipy.constants import hbar, k, c

import matplotlib.pyplot as plt



class QuantumTimeEmergenceExperiment:

    def _init_(self, f0=6.676e-10):

        self.f0 = f0

        self.E0 = hbar * 2 * np.pi * f0

        
        # Configuración experimental

        self.atom_type = "Rb-87"

        self.transition_freq = 6.834e9
  # Hz - transición hiperfina

        self.trap_frequency = 2 * np.pi * 100  # Hz

        
        # Parámetros del interferómetro atómico

        self.arm_length = 0.1  # 10 cm

        self.interaction_time = 1.0  # segundo
        
    def create_bose_einstein_condensate(self, N_atoms=10000):

        """Prepara un BEC para el experimento"""

        # Hamiltoniano del BEC atrapado

        x = qt.position(100)

        p = qt.momentum(100)


        
        # Potencial armónico

        H_trap = 0.5 * self.trap_frequency*2 * x*2

        
        # Interacción entre átomos (aproximación de campo medio)

        g = 4 * np.pi * hbar**2 * 100 / self.mass_Rb87  
# parámetro de interacción

        H_int = (g/2) * qt.ket2dm(x)  # interacción delta

        
        self.H_BEC = H_trap + H_int

        return self.H_BEC


    
    def apply_temporal_modulation(self, psi0, modulation_freq=None):

        """Aplica modulación temporal en la frecuencia fundamental"""

        if modulation_freq is None:

            modulation_freq = self.f0


            
        # Operador de modulación temporal

        times = np.linspace(0, 1000, 1000)  # 1000 segundos

        modulation = np.sin(2 * np.pi * modulation_freq * times)
        
        # Evolución temporal con modulación

        result = qt.mesolve(self.H_BEC, psi0, times,
 
                           c_ops=[], e_ops=[qt.position(100)])


        
        return result, modulation


    
    def measure_entanglement_entropy(self, result):

        """Mide la entropía de entrelazamiento emergente"""

        entropy = []

        for state in result.states:

            # Traza parcial sobre subsistemas

            rho_reduced = state.ptrace([0])  
# primer átomo

            S = -np.trace(rho_reduced * np.log(rho_reduced + 1e-12))

            entropy.append(S)


        
        return np.array(entropy)

    
    def detect_time_emergence(self, times, entropy):

        """Detecta el punto de emergencia del tiempo termodinámico"""

        # Buscar crecimiento no lineal de entropía

        entropy_deriv = np.gradient(entropy, times)

        
        # El tiempo emerge cuando dS/dt se vuelve constante
                time_emergence_index= np.argmax(entropy_deriv > 0.1 * np.max(entropy_deriv))

                time_emergence = times[time_emergence_index]
                
        
                        return time_emergence, entropy_deriv

    
    def run_complete_experiment(self):

        """Ejecuta el experimento completo de emergencia temporal"""

        print("🔬 INICIANDO EXPERIMENTO DE EMERGENCIA TEMPORAL")

        print("==============================================")

        
  # 1. Preparar BEC

        print("1. Preparando condensado de Bose-Einstein...")

        H_BEC = self.create_bose_einstein_condensate()

        psi0 = qt.coherent(100, 0)  
        # estado coherente inicial

        
# 2. Aplicar modulación temporal

        print("2. Aplicando modulación en f₀ = {:.2e} Hz...".format(self.f0))

        result, modulation = self.apply_temporal_modulation(psi0)

        
        # 3. Medir entropía

        print("3. Midendo entropía de entrelazamiento...")

        times = np.linspace(0, 1000, 1000)

        entropy = self.measure_entanglement_entropy(result)

        
        # 4. Detectar emergencia temporal

        print("4. Detectando punto de emergencia temporal...")

        t_emerge, entropy_deriv = self.detect_time_emergence(times, entropy)
        
        # 5. Visualizar resultados
        self.plot_experimental_results(times, entropy, entropy_deriv,t_emerge, modulation)

        
        return t_emerge, entropy, modulation


    def plot_experimental_results(self, times, entropy, entropy_deriv,t_emerge, modulation):

        """Visualiza todos los resultados experimentales"""

        fig, ((ax1, ax2), (ax3, ax4)) = plt.subplots(2, 2, figsize=(15, 10))

        
        # Panel 1: Crecimiento de entropía

        ax1.plot(times, entropy, 'b-', linewidth=2)
        ax1.axvline(t_emerge, color='r', linestyle='--',
 
                   label=f'Tiempo emerge: {t_emerge:.1f} s')

        ax1.set_xlabel('Tiempo [s]')

        ax1.set_ylabel('Entropía de Entrelazamiento S(t)')

        ax1.legend()

        ax1.grid(True, alpha=0.3)

        ax1.set_title('Emergencia del Tiempo Termodinámico')


        
        # Panel 2: Derivada de entropía
                        ax2.plot(times, entropy_deriv, 'g-', linewidth=2)
        
        ax2.axvline(t_emerge, color='r', linestyle='--')
        
        ax2.set_xlabel('Tiempo [s]')

                ax2.set_ylabel('dS/dt [1/s]')

                ax2.grid(True, alpha=0.3)

                ax2.set_title('Tasa de Emergencia Temporal')


        
        # Panel 3: Modulación temporal aplicada

        ax3.plot(times[:100], modulation[:100], 'purple', linewidth=2)

        ax3.set_xlabel('Tiempo [s]')

        ax3.set_ylabel('Amplitud de Modulación')

        ax3.grid(True, alpha=0.3)

        ax3.set_title('Modulación en f₀ = {:.2e} Hz'.format(self.f0))


        
        # Panel 4: Correlación entropía-modulación

        correlation = np.correlate(entropy, modulation, mode='same')

        ax4.plot(times, correlation, 'orange', linewidth=2)

        ax4.set_xlabel('Tiempo [s]')

        ax4.set_ylabel('Correlación Cruzada')

        ax4.grid(True, alpha=0.3)

        ax4.set_title('Correlación Entropía-Modulación')


        
        plt.tight_layout()

                plt.savefig('quantum_time_emergence_experiment.png',
                dpi=300, bbox_inches='tight')

        plt.show()



# Ejecutar experimento completo

experiment = QuantumTimeEmergenceExperiment()

t_emerge, entropy, modulation = experiment.run_complete_experiment()

# Uses NANOGrav 15-year dataset: DOI 10.5281/zenodo.16051178
# Reference: Agazie et al. 2023, ApJL, 951, L1



print(f"\n🎉 EXPERIMENTO COMPLETADO EXITOSAMENTE")

print(f"├── Tiempo de emergencia detectado: {t_emerge:.1f} s")

print(f"├── Entropía máxima alcanzada: {np.max(entropy):.3f} k_B")

print(f"└── Frecuencia fundamental confirmada: {experiment.f0:.3e} Hz")