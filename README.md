# FoudationProjectSwift



## Project

//
//  ContentView.swift
//  Project-pokemon-manha
//
//  Created by Mack Aluno on 29/07/25.
//
import Foundation
import SwiftUI


struct ContentView: View {
    @StateObject private var viewModel = PokemonViewModel()
    
    var body: some View {
        TabView {
            ZStack {
                VStack {
                    Text("POKEMONS")
                        .foregroundColor(.gray)
                        .fontWeight(.bold)
                        .font(.title2)

                    ScrollView(.horizontal, showsIndicators: false) {
                        HStack(spacing: 16) {
                            ForEach(viewModel.pokemons, id: \.name) { pokemon in
                                VStack {
                                    AsyncImage(url: URL(string: pokemon.imageUrl)) { image in
                                        image.resizable()
                                    } placeholder: {
                                        ProgressView()
                                    }
                                    .frame(width: 80, height: 80)
                                    .background(Color.gray.opacity(0.2))
                                    .clipShape(RoundedRectangle(cornerRadius: 10))
                                    
                                    Text(pokemon.name.capitalized)
                                        .font(.caption)
                                }
                            }
                        }
                        .padding()
                    }
                }
            }
            .tabItem { Label("Menu", systemImage: "house") }.tag(1)
            
            Text("Tab Content 2")
                .tabItem { Label("Estatísticas", systemImage: "chart.line.uptrend.xyaxis") }.tag(2)
            
            Text("Tab Content 3")
                .tabItem { Label("Pesquisar", systemImage: "magnifyingglass.circle.fill") }.tag(3)
        }
        .onAppear {
            viewModel.fetchPokemons()
        }
    }
}


import Foundation

struct Pokemon: Codable {
    let name: String
    let url: String
    
    var id: Int {
        // Pega o ID do final da URL (ex: "https://pokeapi.co/api/v2/pokemon/1/")
        Int(url.split(separator: "/").last ?? "0") ?? 0
    }
    
    var imageUrl: String {
        "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/\(id).png"
    }
}


import Foundation
import Combine

class PokemonViewModel: ObservableObject {
    @Published var pokemons: [Pokemon] = []

    func fetchPokemons() {
        guard let url = URL(string: "https://pokeapi.co/api/v2/pokemon?limit=10") else { return }

        URLSession.shared.dataTask(with: url) { data, _, error in
            guard let data = data, error == nil else { return }

            do {
                let decodedResponse = try JSONDecoder().decode(PokemonResponse.self, from: data)
                DispatchQueue.main.async {
                    self.pokemons = decodedResponse.results
                }
            } catch {
                print("Erro ao decodificar:", error)
            }
        }.resume()
    }
}

struct PokemonResponse: Codable {
    let results: [Pokemon]
}



## FUNCIONAMENTO DO FLUXO:

O ContentView aparece.

Chama viewModel.fetchPokemons().

A função busca a lista na PokéAPI.

A API retorna um JSON com nomes e URLs.

A ViewModel cria objetos Pokemon.

A interface mostra a imagem e nome de cada um.

 
## Organização de Arquivos:

Pokemon.swift: struct do modelo.

PokemonViewModel.swift: lógica da API.

ContentView.swift: interface com os dados.

