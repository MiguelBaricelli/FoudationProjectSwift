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


// SwiftUI Pokémon App com estatísticas e captura

import SwiftUI

## Models
struct Pokemon: Identifiable, Codable, Hashable {
    let id = UUID()
    let name: String
    let url: String
    var imageUrl: String {
        let index = url.split(separator: "/").dropLast().last ?? "1"
        return "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/\(index).png"
    }
    var number: Int {
        return Int(url.split(separator: "/").dropLast().last ?? "1") ?? 1
    }
}

struct PokemonDetail: Codable {
    let types: [TypeSlot]
    let stats: [Stat]
}

struct TypeSlot: Codable {
    let type: NamedAPIResource
}

struct Stat: Codable {
    let base_stat: Int
    let stat: NamedAPIResource
}

struct NamedAPIResource: Codable {
    let name: String
}

## ViewModel
class PokemonViewModel: ObservableObject {
    @Published var allPokemon: [Pokemon] = []
    @Published var capturedPokemon: [Pokemon] = []
    @Published var capturedDetails: [Pokemon: PokemonDetail] = [:]

    init() {
        fetchPokemon()
    }

    func fetchPokemon() {
        guard let url = URL(string: "https://pokeapi.co/api/v2/pokemon?limit=151") else { return }

        URLSession.shared.dataTask(with: url) { data, _, _ in
            guard let data = data else { return }
            if let decoded = try? JSONDecoder().decode(PokemonListResponse.self, from: data) {
                DispatchQueue.main.async {
                    self.allPokemon = decoded.results
                }
            }
        }.resume()
    }

    func fetchDetail(for pokemon: Pokemon, completion: @escaping (PokemonDetail) -> Void) {
        guard let url = URL(string: pokemon.url) else { return }

        URLSession.shared.dataTask(with: url) { data, _, _ in
            guard let data = data else { return }
            if let detail = try? JSONDecoder().decode(PokemonDetail.self, from: data) {
                DispatchQueue.main.async {
                    self.capturedDetails[pokemon] = detail
                    completion(detail)
                }
            }
        }.resume()
    }

    func toggleCapture(_ pokemon: Pokemon) {
        if capturedPokemon.contains(pokemon) {
            capturedPokemon.removeAll { $0 == pokemon }
            capturedDetails.removeValue(forKey: pokemon)
        } else {
            capturedPokemon.append(pokemon)
            fetchDetail(for: pokemon) { _ in }
        }
    }

    func totalByType() -> [String: Int] {
        var count: [String: Int] = [:]
        for detail in capturedDetails.values {
            for type in detail.types {
                count[type.type.name, default: 0] += 1
            }
        }
        return count
    }

    func strongestPokemon() -> (Pokemon, Int)? {
        return capturedDetails.max { a, b in
            totalStats(for: a.value) < totalStats(for: b.value)
        }.map { ($0.key, totalStats(for: $0.value)) }
    }

    func weakestPokemon() -> (Pokemon, Int)? {
        return capturedDetails.min { a, b in
            totalStats(for: a.value) < totalStats(for: b.value)
        }.map { ($0.key, totalStats(for: $0.value)) }
    }

    private func totalStats(for detail: PokemonDetail) -> Int {
        detail.stats.map { $0.base_stat }.reduce(0, +)
    }
}

struct PokemonListResponse: Codable {
    let results: [Pokemon]
}

## Views
struct ContentView: View {
    @StateObject private var viewModel = PokemonViewModel()
    @State private var selection = 0

    var body: some View {
        TabView(selection: $selection) {
            PokemonListView(viewModel: viewModel, selection: $selection)
                .tabItem { Label("Pokémons", systemImage: "list.bullet") }
                .tag(0)

            StatisticsView(viewModel: viewModel, selection: $selection)
                .tabItem { Label("Estatísticas", systemImage: "chart.bar") }
                .tag(1)
        }
    }
}

struct PokemonListView: View {
    @ObservedObject var viewModel: PokemonViewModel
    @Binding var selection: Int

    var body: some View {
        NavigationView {
            VStack {
                HStack {
                    Spacer()
                    Button("Ver Estatísticas") {
                        selection = 1
                    }.padding()
                }
                Text("POKEMONS")
                    .font(.largeTitle)
                    .bold()
                ScrollView {
                    LazyVStack {
                        ForEach(viewModel.allPokemon) { pokemon in
                            HStack {
                                AsyncImage(url: URL(string: pokemon.imageUrl)) { image in
                                    image.resizable().scaledToFit().frame(width: 50, height: 50)
                                } placeholder: {
                                    ProgressView()
                                }
                                Text(pokemon.name.capitalized)
                                    .font(.title3)
                                Spacer()
                            }
                            .padding()
                            .background(viewModel.capturedPokemon.contains(pokemon) ? Color.green.opacity(0.3) : Color.clear)
                            .cornerRadius(10)
                            .onTapGesture {
                                viewModel.toggleCapture(pokemon)
                            }
                        }
                    }
                }
            }
            .navigationBarHidden(true)
        }
    }
}

struct StatisticsView: View {
    @ObservedObject var viewModel: PokemonViewModel
    @Binding var selection: Int

    var body: some View {
        NavigationView {
            VStack(alignment: .leading, spacing: 16) {
                HStack {
                    Spacer()
                    Button("Voltar") {
                        selection = 0
                    }.padding()
                }
                Text("Estatísticas")
                    .font(.largeTitle)
                    .bold()
                Text("Total capturados: \(viewModel.capturedPokemon.count)")
                Text("Faltam: \(151 - viewModel.capturedPokemon.count)")
                ForEach(Array(viewModel.totalByType().sorted { $0.key < $1.key }), id: \.(key)) { type, count in
                    Text("\(type.capitalized): \(count)")
                }
                if let strongest = viewModel.strongestPokemon() {
                    Text("Mais forte: \(strongest.0.name.capitalized) com \(strongest.1) pontos")
                }
                if let weakest = viewModel.weakestPokemon() {
                    Text("Mais fraco: \(weakest.0.name.capitalized) com \(weakest.1) pontos")
                }
                Spacer()
            }
            .padding()
            .navigationBarHidden(true)
        }
    }
}

## Preview
@main
struct PokemonApp: App {
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
    }
}


PokemonViewModel.swift: lógica da API.

ContentView.swift: interface com os dados.

